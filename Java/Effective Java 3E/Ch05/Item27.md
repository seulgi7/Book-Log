# 아이템 27. 비검사 경고를 제거하라

> 비검사(unckecked) 경고(warning)
>
> - 컴파일러가 타입 안전성을 확인하는데 필요한 정보가 충분치 않을 때 발생시키는 경고.

- 할 수있는 한 모든 비검사 경고를 제거하라!

  - 제네릭을 사용하기 시작할 때 볼 수 있는 수많은 컴파일러  경고를 보게 될 것이다.

    - 비검사 형변환 경고, 비검사 메서드 호출 경고, 비검사 매개변수화 가변인수 타입 경고, 비검사 변환 경고 등.

  - 모두 제거한다면 그 코드는 타입 안전성이 보장된다.

    ➣ 즉, 런타임에 classCastException이 발생할 일이 없고, 개발자가 의도한대로 잘 동작하리라 확신할 수 있다.

  - 예시

    - 잘못 작성된 코드

      ```java
      Set<Lark> exaltation = new HashSet();
      ```
      

      ![iem27_warning](https://github.com/seulgi7/Book-Log/blob/cbdb7921da603acfb440c0fbb680ebd698010384/Java/Effective%20Java%203E/iamge/iem27_warning.png)

    - 경고를 제거한 코드

      ```java
      Set<Lark> exaltion = new HashSet<>();
      ```

      - 자바 7부터 지원하는 다이아몬드 연산자(<>)만으로 해결할 수 있다.
        - 컴파일러가 올바른 실제 타입 매개변수를 추론해줌.

## @SuppressWarnings("unchecked") 에너테이션

- **경고를 제거할 수는 없지만 타입 안전하다고 확신할 수 있다면 @SuppressWarnings("unchecked") 에너테이션을 달아 경고를 숨기자.**

  - 주의점

    - 타입 안전성을 검증하지 않은 채 경고를 숨기면 스스로에게 잘못된 보안 인식을 심어주는 꼴이다. 그 코드는 경고 없이 컴파일 되겠지만, 런타임에는 여전히 ClassCastException을 던질 수 있다.
    - 안전하다고 검증된 비검사 경고를 (숨기지 않고) 그대로 두면, 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있다. 제거하지 않은 수많은 거짓 경고 속에 새로운 경고가 파묻힐 것이기 때문이다.

  - **@SuppressWarnings 애노테이션은 항상 가능한 한 좁은 범위에 적용하자. **

    - 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다.
      - 단, return 문에는 @SuppressWarnings을 다는게 불가능하다.[JLS, 9.7]
    - 보통은 변수 선언, 아주 짧은 메서드, 혹은 생성자가 될것이다.
    - 자칫 심각한 경고를 놓칠 수 있으니 절대로 클래스 전체에 적용해서는 안된다.

  - **@SuppressWarnings 애노테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다.**

  - 코드

    - before

      ![item27_27-1_before](https://github.com/seulgi7/Book-Log/blob/cbdb7921da603acfb440c0fbb680ebd698010384/Java/Effective%20Java%203E/iamge/item27_27-1_before.png)

    - after

      ```java
      public class ListExmaple {
          private int size;
          Object[] elements;
      
          public <T> T[] toArray(T[] a){
            if(a.length < size){
              //생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
            	@SupressWarnings("unchecked")
              T[] result = (T[]) Arrays.copyOf(elements, size, a.getClass());
              return result;
            }
            System.arraycopy(elements, 0, a, 0, size);
            if(a.length > size) a[size] = nulll;
            return  a;
          }
      }
      ```

      
