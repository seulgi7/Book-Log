# Item70. 복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라.


## 예외의 4가지 종류

예외는 throwable 타입이라고 한다.
예외는 검사, 비검사 예외가 있으며 하위 예외로 런타임 예외와 에러가 있고 각각의 목적이 있다.

1. 검사 예외 (Checked Exception)
2. 검사 예외 (Checked Exception)
   1. 런타임 예외 (Runtime Exception)
   2. 에러 (Error)

검사 예외와 비검사 예외는 예외 처리를 강제하냐 아니냐의 차이가 있다.



## 예외별 사용 지침

### 검사 예외

- 호출하는 쪽에서 복구하리라 여겨지는 상황에 사용한다.
  - 검사 예외를 던지면 호출자가 그 예외를 catch로 잡아 처리하거나 더 바깥으로 전파하도록 강제하게 된다.
    즉, **메서드 선언에 포함된 검사 예외** 각각은 **해당 메서드를 호출했을 때 발생할 수 있는 결과**이므로 API 사용자에게 회복하라고 요구하는 것이다.
- catch 로 복구하거나 밖으로 퍼트려(throws) 복구할 가능성이 있다.
  - 복구할 가능성이 있으므로, 복구에 도움이 되는 여러 메서드들을 제공해주면 좋다.(아이템75)
  - ex) 통장 잔고가 부족하다면, 얼마가 부족한지 알려주면 예외 복구에 도움이 된다. ATM기에서 그냥 잔액이 부족하다고 알리기보다 만원을 인출하기 위해서 3천원이 모자르다고 알려주면, 고객은 3천원을 더 입금하고 만원을 출금할 수 있다.

---

### 비검사 예외

#### 비검사 throwable(런타임 예외, 에러)의 공통점

- 프로그램에서 잡을 필요가 없거나 혹은 통상적으로 잡지 말아야 한다.
- 프로그램에서 해당 예외를 던졌다는 것은 **복구가 불가능하거나 더 실행해봐야 득보다는 실이 많다**는 뜻이다.
  만약, **비검사 예외를 잡지 않은 스레드는 오류 메시지를 내뱉으며 중단**된다.

#### 런타임 에러

- **프로그래밍 오류를 나타낼 때**

  - 클라이언트가 해당 **API의 명세에 기록된 제약을 지키지 못했을 때**는 런타임 예외를 발생시키자.(즉, 프로그래밍 오류)
    ex) ArrayIndexOutOfBoundsException
    배열의 인덱스는 0에서 배열 크기 -1 사이여야 한다.는 전제 조건이 지켜지지 않았다는 뜻이다.

    ```java
    public class Car {
    	private String name;
    
    	public Car(String name) {
    		if(name.length() > 5){
    			throw new IllegalArgumentException("이름은 5자를 넘을 수 없습니다.");
    		}
    		this.name = name;
    	}
    }
    ```



---

### 복구할 수 있는 상황인지 프로그래밍 오류인지 명확히 구분되지 않는다!

예를 들어 자원 고갈이 말도 안 되는 크기의 배열을 할당해 생긴 프로그래밍 오류일 수도 있고, 지원이 일시적으로 부족한 것일수도 있다.(수요가 몰려서) 이 상황 같은 경우는 복구할 수 있는 상황이다.

따라서 해당 **예외가 복구될 수 있는 것인지는 API 설계자의 판단에 달렸다.**

> 복구 가능하다면 검사 예외를, 그렇지 않다면 런타임 예외를 던지자.
> 확신하기 어렵다면, 비검사 예외를 던지는게 나을 것이다.(아이템 71)

---

### 에러를 사용하는 상황

보통 JVM이 자원 부족 등 더 이상 수행을 할 수 없는 상황을 나타낸다.
특히나, **Error 클래스를 상속해 하위 클래스를 만드는 일은 자제하자**

즉, 우리가 구현하는 비검사 예외는 모두 RuntimeException의 하위 클래스여야 한다.
(AssertionError 제외)

---

## 핵심정리

> 1. 복구할 수 있는 상황이라면 검사 예외를 사용하자.
> 2. 프로그래밍 오류라면 비검사 예외를 사용하자
> 3. 복구할 수 있는 상황인지 프로그래밍 오류인지 확실하지 않다면 비검사 예외를 던지자
> 4. 검사 예외라면 예외를 벗어나는데 정보를 알려주는 접근자 메서드(아이템 75)도 같이 제공하자