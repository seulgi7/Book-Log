# Override 애너테이션

------

💡 자바가 기본으로 제공하는 애너테이션중 가장 중요한 것은 @Override

- 이 애너테이션은 상위 타입의 메서드를 재정의 했음을 의미

> equals와 hashCode 재정의

------

```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }

    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }

    public int hashCode() {
        return 31 * first + second;
    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++) {
            for (char ch = 'a'; ch <= 'z'; ch++) {
                s.add(new Bigram(ch, ch));
            }
        }
        System.out.println(s.size());
    }
}
```

- 26이 나와야 할 것 같지만 실제로는 260이 출력
    - equals를 재정의(overriding)한 게 아니라 '다중정의(overloading,아이템52)'해버림.
    - **Object의 equals를 재정의하려면 매개변수 타입을 Object**로 해야만 하는데, 그렇게 하지 않은 것.
    - Object에서 상속한 equals와는 별개인 equals를 새로 정의한 꼴이 됨.
    - **Object의 equals는 ==연산자와 똑같이 객체 식별성(identity)만을 확인한다.**
    - 따라서 같은 소문자를 소유한 바이그램 10개 각가이 서로 다른 객체로 인식되고, 결국 260을 출력한 것.
  

> Override를 이용해서 컴파일시 확인 가능

------

```java
@Override
public boolean equals(Bigram b){
    return b.first == first && b.second==second;
}
----
=> Method does not override method from its superclass
```

- @Override 애너테이션을 사용하면 컴파일 오류 발생

> 수정 코드

------

```java
@Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Bigram bigram = (Bigram) o;
        return first == bigram.first &&
                second == bigram.second;
    }
```

💡 이러한 실수를 하지 않기 위해 **상위 클래서의 메서드를 재정의하려는 모든 메서드에 @Override 애너테이션을 달자!**

### 예외

------
- 예외는 한 가지 뿐.
  - **구체 클래스에서 상위 클래스의 추상 메서드를 재정의할 경우**
    - 구체 클래스인데 아직 구현하지 않은 추상 메서드가 남아 있다면 컴파일러가 알려주기 때문.
    - IDE에서는 설정 적용시 @Override가 달려 있지 메서드가 실제로는 재정의를 했다면 경고
  

### @Override의 또 다른 역할 : 인터페이스 메서드를 재정의

------

- @Override는 클래스뿐 아니라 인터페이스의 메서드를 재정의할 때도 사용
- 인터페이스 메서드를 구현한 메서드에서도 @Override를 다는 습관을 들이면 시그니처가 올바른지 재차 확신 가능

```java
public interface itemInterface {
    public void testItem();
}
---------------------------------------------------------

public class ItemClass implements itemInterface{
        // Override를 달아서 확인
    @Override
    public void testItem() {
        System.out.println("인터페이스 재정의");
    }
}
```

### 결론

------

- 재정의한 모든 메서드에 @Override 애너테이션을 의식적으로 달면 실수 했을 경우 컴파일 오류 확인 가능
- 예외는 구체 클래스에서 상위 클래스의 추상 메서드를 재정의한 경우 ( 이 경우에 달아도 문제는 없다. )
