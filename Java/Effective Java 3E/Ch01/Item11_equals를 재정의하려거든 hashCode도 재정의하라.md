## 아이템 11. equals를 재정의하려거든 hashCode도 재정의하라

**equals를 재정의한 클래스 모두에서 hashCode도 재정의해야 한다.**

- 왜? 그렇지 않으면 hashCode 일반 규약을 어기게 되어 해당 클래스의 인스턴스를 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다.

- HashCode 재정의를 잘못했을 때 크게 문제가 되는 조항은 아래 두번째다. 즉, **논리적으로 같은 객체는 같은 해시코드를 반환해야 한다.**

  > [ Object 명세에서 발췌한  규약]
  >
  > 1. equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 HashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
  >
  >    단, 애플리케이션을 다시 실행한다면 이 값이 달라져도 상관없다.
  >
  > 2. **equals(Object)가 두 객체가 같다고 판단했다면, 두 객체의 HashCode는 똑같은 값을 반환해야 한다.**
  >
  > 3. equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 HashCode가 서로 다른 값을 반환할 필요는 없다. 
  >
  >    단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

  - 아이템10의 equals는 물리적으로 다른 객체를 논리적으로 같다고 할 수 있다. 하지만 Object의 기본 HashCode 메서드는 이 둘이 전혀 다르다고 판단하여, 규약과 달리 (무작위처럼 보이는) 서로 다른 값을 반환한다.

    ```java
    Map<PhoneNumber, String> m = new HashMap<>();
    m.put(new PhoneNumber(707, 867, 5309), "제니");
    m.get(new PhoneNumber(707, 867, 5309));//null
    ```

    - 왜? PhoneNumber클래스는 hashCode를 재정의하지 않았기 때문에 논리적인 동치인 두 객체가 서로 다른 해시코드를 반환하여 두 번째 규약을 지키지 못한다.

    - 해결방법

      1. 최악의 (하지만 적합한) hashCode 구현 - 사용금지!

         ```java
         @Override
         public int hashCode( return 42 );
         ```

         - 동치인 모든 객체에서 똑같은 해시코드를 반환하니 적법하다.

         - 최악의 구현방법인 이유

           - 모든 객체에게 똑같은 값만 내어주므로 모든 객체가 해시테이블의 버킷 하나에 담겨 마치 연결 리스트(linked list)처럼 동작한다.

             ➣ 평균 수행 시간 O(1) → O(n) ; 객체가 많아지면 도저히 쓸 수 없게 됨.

           - 3번째 규약이 요구하는 속성대로 좋은 해시 함수라면 서로 다른 인스턴스에 다른 해시코드를 반환한다. 

      2. 좋은 hashCode를 작성하는 간단한 요령.

         ```java
         // equals를 재정의하면 hashCode로 재정의해야 함을 보여준다. (70-71쪽)
         public final class PhoneNumber {
             private final short areaCode, prefix, lineNum;
         
             public PhoneNumber(int areaCode, int prefix, int lineNum) {
                 this.areaCode = rangeCheck(areaCode, 999, "area code");
                 this.prefix   = rangeCheck(prefix,   999, "prefix");
                 this.lineNum  = rangeCheck(lineNum, 9999, "line num");
             }
         
             private static short rangeCheck(int val, int max, String arg) {
                 if (val < 0 || val > max)
                     throw new IllegalArgumentException(arg + ": " + val);
                 return (short) val;
             }
         
             @Override public boolean equals(Object o) {
                 if (o == this)
                     return true;
                 if (!(o instanceof PhoneNumber))
                     return false;
                 PhoneNumber pn = (PhoneNumber)o;
                 return pn.lineNum == lineNum && pn.prefix == prefix
                         && pn.areaCode == areaCode;
             }
         
         
            /* 코드 11-2 전형적인 hashCode 메서드 (70쪽) */
            @Override public int hashCode() {
                 int result = Short.hashCode(areaCode); // 1
                 result = 31 * result + Short.hashCode(prefix); // 2
                 result = 31 * result + Short.hashCode(lineNum); // 3
                 return result;
             }
         
             public static void main(String[] args) {
                 Map<PhoneNumber, String> m = new HashMap<>();
                 m.put(new PhoneNumber(707, 867, 5309), "제니");
                 System.out.println(m.get(new PhoneNumber(707, 867, 5309)));
             }
         }
         ```

         1. int 변수 result를 선언한 후 값 c로 초기화한다.

            이때 c는 해당 객체의 첫번째 핵심 필드를 단계 2.a방식으로 계산한 해시코드이다.

            ( 여기서 핵심필드란 equals 비교에 사용되는 필드를 말한다. (참조 :아이템10) )

         2. 해당 객체의 나머지 핵심 필드 f 각각에 대해 다음 작업을 수행한다.

            a. 해당 필드의 핵심코드 c를 계산한다.
              1. 기본 타입 필드라면, Type.hashCode(f)를 수행한다. 여기서 Type은 해당 기본 타입의 박싱 클래스다.
              2. 참조 타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals를 재귀적으로 호출해 비교한다면, 이 필드의 hashCode를 재귀적으로 호출한다.
                 계산이 더 복잡해질 것 같으면, 이 필드의 표준형(ca-nonical representation)을 만들어 그 표준형의 hashCode를 호출한다.
                 필드의 값이 null이면 0을 사용한다.(다른 상수도 괜찮지만 전통적으로 0을 사용한다.)
              4. 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.
                 이상의 규칙을 재귀적으로 적용해 각 핵심 원소의 해시코드를 계산한 다음, 단계 2.b방식으로 갱신한다.
                 배열에 핵심원소가 없다면 단순히 상수를 사용한다.(0을 추천한다.)
                 모든 원소가 핵심 원소라면 Arrays.hashCode를 사용한다.

            b. 단계 2.a에서 계산한 해시코드 c로 result를 갱신한다.
              ```java
              result = 31 * result + c;
              ```
               - 곱할 숫자를 31로 정한 이유는 31이 홀수이면서 소수(prime)이기 때문이다.

                 소수를 사용하는 이점은 그다지 분명하지 않지만 전통적으로 널리 사용된다. 31의 좋은점은 곱셈을 시프트와 뺄셈의 조합으로 바꾸면 더 좋은 성능을 낼 수 있다는 것이다.
     
         3.  result를 반환한다.

      


      3. Object 클래스의 hash메서드 사용

         ```java
         /* 코드 11-3 한 줄짜리 hashCode메서드 - 성능이 살짝 아쉽다. */
         @Override public int hashCode(){
           return Objects.hash(lineNum, prefix, areaCode);
         }
         ```

         - Object 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 hash를 제공한다. 
         - 단점 : 속도는 더 느림
           - 왜? 입력 인수를 담기위한 배열이 만들어지고, 입력 중 기본타입이 있다면 박싱과 언박싱도 가져야하기 때문
         - 성능에 민감하지 않은  상황에서만 사용하자.

      4.  캐싱하는 방식

         ```java
         /* 코드 11-4 해시코드를 지연 초기화하는 hashCode 메서드 - 스레드 안정성까지 고려해야 한다.*/
         private int hashCode; //자동으로 0으로 초기화한다.
         
         @Override public int hashCode(){
           int result = hashCode;
           if(result == 0){
           	result = Short.hashCode(areaCode); // 1
           	result = 31 * result + Short.hashCode(prefix); // 2
           	result = 31 * result + Short.hashCode(lineNum); // 3
           	hashCOde = result;
           }
           return result;
         }
         ```

         - 클래스가 불변이고 해시코드를 계산하는 비용이 큰 경우 매번 새로 계산하기 보다는 캐싱하는 방식을 고려해야 한다.
         - 주의점 : 스레드를 안전하게 만들도록 신경써야 한다.(아이템83)
           - 여러개의 스레드가 동시에 들어온 경우, 엇갈리면서 계산하다보면  동일한 값을 가진 불변 객체인데 해시코드가 다른 상황이 발생할 수도 있다.

    - 주의점

      - 파생  필드는 해시코드 계산에서 제외해도 된다.

        즉, 다른 필드로부터 계산해낼 수 있는 필드는 모두 무시해도 된다.

      - equals 비교에 사용되지 않은 필드는 '반드시' 제외해야 한다.

      - 성능을 높인답시고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다.

      - hashCode가 반환하는 값의 생성규칙을 API 사용자에게 자세히 공표하지 말자. 그래야 클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있다.

        - 자세한 규칙을 공표하지 않는다면, 해시 기능에서 결함을 발견했거나 더 나은 해시 방식을 알아낸 경우 다음 릴리스에서 수정할 수 있다.

> 핵심정리
>
> equals를 재정의할 때는 hashCode도 재정의해야한다. 그렇지 않으면 프로그램이 제대로 동작하지 않을 것이다.
>
> 재정의한 hashCode는 Object의 API문서에 기술된 일반 규약을 따라야 하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야 한다.
>
> 이렇게 구현하기가 어렵지는 않지만 조금 따분한 일이긴하다. (68의 요령 참조)
>
> 하지만 걱정하지마시라. 아이템 10에서 이야기한 AutoValue 프레임워크를 사용하면 멋진 equals와 hashCode를 자동으로 만들어준다. IDE들도 이런 기능을 일부 제공한다.

