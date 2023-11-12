## 동시성 프로그래밍의 역사
### 1. 릴리스
- 스레드, 동기화, wait/notify를 지원했다.

- wait : 갖고 있던 고유 락을 해제하고, 스레드를 잠들게 한다.
- notify :잠들어 있던 스레드 중 임의로 하나를 골라 깨운다.
### 2. 자바 5 이후
동시성 컬렉션인 java.util.concurrent 라이브러리와 실행자(Executor) 프레임워크 지원했다.

### 3. 자바 7 이후
고성능 병렬 분해 프레임워크인 포크-조인(fork-join) 패키지를 추가했다.

### 4. 자바 8 이후
parallel 메서드만 한 번 호출하면 파이프라인을 병렬 실행할 수 있는 스트림을 지원했다.

이처럼 동시성 프로그램을 작성하기 점점 쉬워지고 있지만, 동시성 프로그래밍을 할때는 항상 안전성(safety)와 응답 가능(liveness) 상태를 유지 해야 하는 것에 주의해야 한다.

## 동시성 프로그래밍 주의점
### 1. 스트림 병렬화를 사용하면 안되는 경우
```java
public class ParallelMersennePrimes {
    public static void main(String[] args) {
        primes().map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
                .parallel() // 스트림 병렬화
                .filter(mersenne -> mersenne.isProbablePrime(50))
                .limit(20)
                .forEach(System.out::println);
    }

    static Stream<BigInteger> primes() {
        return Stream.iterate(TWO, BigInteger::nextProbablePrime);
    }
}
```
무작정 성능을 향상시키기 위해 parallel() 를 사용하면, 위와 같이 아무것도 출력하지 못하면서 CPU는 90% 나 잡아먹는 상태가 무한히 계속되는 문제가 발생할 수 있다. 
- 왜 ?
    - 스트림 라이브러리가 이 파이프라인을 병렬화하는 방법을 찾아내지 못했기 때문이다.
      > **파이프라인 병렬화로 성능 개선을 할 수 없는 경우**
      > 1. 데이터 소스가 Stream.iterate인 경우
      > 2. 중간 연산으로 limit 을 사용하는 경우
    -  파이프라인 병렬화는 limit를 다룰 때 CPU 코어가 남는다면 원소를 몇 개 더 처리한 후 제한된 개수 이후의 결과를 버려도 아무런 해가 없다고 가정한다.
        -  그런데 이 코드의 경우 새롭게 메르센 소수를 찾을 때마다 그 전 소수를 찾을 때보다 두 배 정도 더 오래 걸린다.
        -  즉, 원소 하나를 계산하는 비용이 대략 그 이전까지의 원소 전부를 계산한 비용을 합친 것만큼 든다는 뜻이다.



## 스트림 병렬화는 어떤 경우 사용해야 할까
**스트림의 소스가 ArrayList, HashMap, HashSet, ConcurrentHashMap의 인스턴스거나 배열, int 범위, long 범위일 때 병렬화의 효과가 가장 좋다.**

- 이 자료구조들은 모두 데이터를 원하는 크기로 정확하고 손쉽게 나눌 수 있어서 일을 다수의 스레드에 분배하기에 좋다는 특징이 있다.
    - 나누는 작업은 Spliterator가 담당하며, Spliterator 객체는 Stream이나 Iterable의 spliterator 메서드로 얻어올 수 있다.
- 또한 이 자료구조들은 원소들을 순차적으로 실행할 때의 참조 지역성(locality of reference)이 뛰어나다.
    - 이웃한 원소의 참조들이 메모리에 연속해서 저장되어 있다는 뜻이다.
    - 참조 지역성이 낮으면 스레는 데이터가 주 메모리에서 캐시 메모리로 전송되어 오기를 기다리며 대부분 시간을 멍하게 보낸다.
    - 참조 지역성은 다량의 데이터를 처리하는 벌크 연산을 병렬화할 때 아주 중요한 요소로 작용한다.
    - 참조 지역성이 가장 뛰어난 자료구조는 기본 타입의 배열이다.
      
**스트림 파이프라인의 종단 연산의 동작 방식 역시 병렬 수행 효율에 영향을 준다.**

- 종단 연산에서 수행하는 작업량이 파이프라인 전체 작업에서 상당 비중을 차지하면서 순차적인 연산이라면 파이프라인 병렬 수행의 효과는 제한될 수 밖에 없다.
- 종단 연산 중 병렬화에 가장 적합한 것은 축소(reduction)다.
    - 축소는 파이프라인에서 만들어진 모든 원소를 하나로 합치는 작업으로, Stream의 reduce 메서드 중 하나, 혹은 min, max, count, sum 같이 완성된 형태로 제공되는 메서드 중 하나를 선택해 수행한다.
    - anyMatch, allMatch, noneMatch처럼 조건에 맞으면 바로 반환되는 메서드도 병렬화에 적합하다.
- 가변 축소(mutable reduction)를 수행하는 Stream의 collect 메서드는 병렬화에 적합하지 않다.
    - 컬렉션들을 합치는 부담이 크기 때문이다.

## HOW?
Stream 명세는 이때 사용되는 함수 객체에 관한 엄중한 규약을 정의해놨다.

예컨대 Stream의 reduce 연산에 건네지는 accumulator(누적기)와 combiner(결합기) 함수는 반드시 결합법칙을 만족하고(associative), 간섭받지 않고(non-interfering), 상태를 갖지 않아야(stateless) 한다.
조건이 잘 갖춰지면 parallel 메서드 호출 하나로 거의 프로세서 코어 수에 비례하는 성능 향상을 만끽할 수 있다.

``` java
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
        .mapToObj(BigInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}
//위 코드는 π(n), 즉 n보다 작거나 같은 소수의 개수를 계산하는 함수다.
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
        .parallel()
        .mapToObj(BigInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}
```
(저자 기준, 쿼드 코어) 위의 코드로 π(10^8)을 계산하는 31초, 아래 코드로 9.2초가 걸렸다.
무작위 수들로 이뤄진 스트림을 병렬화하려거든 ThreadLocalRandom(혹은 구식인 Random)보다는 SplittableRandom 인스턴스를 이용하자.

SplittableRandom은 정확히 이럴 때 쓰고자 설계된 것이라 병렬화하면 성능이 선형으로 증가한다.
ThreadLocalRandom은 단일 스레드에서 쓰고자 만들어졌다.
Random은 모든 연산을 동기화하기 때문에 병렬 처리하면 성능이 최악일 것이다.
