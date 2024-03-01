# 아이템 89. 인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라.
## 싱글턴 패턴의 직렬화
이 클래스는 바깥에서 생성자를 호출하지 못하게 막는 방식으로, 인스턴스가 오직 하나만 만들어짐을 보장했다.
```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { }

    public void leaveTheBuilding() { ... }
}
```

- **이 클래스는 선언에 implements Serializable을 추가하는 순간 더이상 싱글턴이 아니게된다.**(아이템3)
  >앞선 아이템 3에서는 아래와 같은 싱글턴 패턴 예제를 보았다. public static final 필드를 사용하는 방식이다. 생성자는 private 접근 지정자로 선언하여 외부로부터 감추고 INSTANCE를 초기화할 때 딱 한 번만 호출된다.
  -  기본 직렬화를 쓰지 않더라도(아이템87), 그리고 명시적인 readObject를 제공하더라도(아이템88) 이 클래스가 초기화될 때 만들어진 인스턴스와는 별개인 인스턴스를 반환하게된다.

## readResolve 기능

이때 readResolve 메서드를 이용하면 readObject 메서드가 만든 인스턴스를 다른 것으로 대체할 수 있다.
이때 readObject 가 만들어낸 인스턴스는 가비지 컬렉션의 대상이 된다.
```java
private Object readResolve() {
    // 기존에 생성된 인스턴스를 반환한다.
    return INSTANCE;
}
```
한편 여기서 살펴본 Elvis 인스턴스의 직렬화 형태는 아무런 실 데이터를 가질 필요가 없으니 모든 인스턴스 필드는 transient 로 선언해야 한다. 
그러니까 readResolve 메서드를 인스턴스의 통제 목적으로 이용한다면 모든 필드는 transient로 선언해야 한다.
만일 그렇지 않으면 역직렬화(Deserialization) 과정에서 역직렬화된 인스턴스를 가져올 수 있다. 
즉, 싱글턴이 깨지게 된다.

## 해결책은 enum
하지만 enum을 사용하면 모든 것이 해결된다. 자바가 선언한 상수 외에 다른 객체가 없음을 보장해주기 때문이다. 
```java
public enum Elvis {
    INSTANCE;
    
    ...필요한 데이터들
}
```
물론 인스턴스 통제를 위해 readResolve 메서드를 사용하는 것이 중요할 때도 있다.
직렬화 가능 인스턴스 통제 클래스를 작성해야 할 때, 컴파일 타임에는 어떤 인스턴스들이 있는지 모를 수 있다.
이   때는 열거 타입으로 표현하는 것이 불가능하기 때문에 readResolve 메서드를 사용할 수 밖에 없다.
