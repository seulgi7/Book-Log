## 아이템17. 변경 가능성을 최소화하라.

### 불변 클래스

- 인스턴스의 내부 값을 수정할 수 없는 클래스.
- 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않음.
- 자바 플랫폼 라이브러리에도 다양한 불변 클래스가 있음.
  - String, 기본타입의 박싱된 클래스들, BigInteger, BigDemical
- 장점 : 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 훨씬 안점함.

> 기본 타입의 박싱 클래스
>
> | 기본형  | 포장 클래스 | 생성 예                           |
> | ------- | ----------- | --------------------------------- |
> | boolean | Boolean     | Boolean bA = new Boolean(true);   |
> | char    | Character   | Character cA = new Chracter("a"); |
> | byte    | Byte        | Byte byA = new Byte(10);          |
> | short   | Short       | Short sA = new Short(1234);       |
> | int     | Integer     | Integer iA = new Integer(1234);   |
> | long    | Long        | Long lA = new Long(1234);         |
> | float   | Float       | Float fA = new Float(12.34f);     |
> | double  | Double      | Double dA = new Double(12.34);    |

### 분별 클래스를 만들기위한 5가지 규칙

1. 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다.
2. 클래스를 확장할 수 없도록 한다.

  - 하위 클래스에서 부주의하게 혹은 나쁜 의도로 객체의 상태를 변하게 만드는 사태를 막아줌.
  - 상속을 막는 대표적인 방법 : 클래스를 final로 선언.

3. 모든 필드를 final로 선언한다.

  - 시스템이 강제하는 수단을 이용해 설계자의 의도를 명확히 드러내는 방법.
  - 새로 생성된 인스턴스를 동기화없이 다른 스레드로 건네도 문제없이 동작하게끔 보장하는 데도 필요함.
    - [자바 언어 명세의 메모리 모델부분(JLS, 17.5;Geotz06,16)](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.5)에 자세히 설명되어 있음

4. 모든 필드를 private으로 선언한다.

  - 필드가 참조하는 가변 객체를 클라이언트에서 직접 접근해 수정하는 일을 막아줌.
  - 기술적으로 기본 타입 필드나 불변 객체를 참조하는 필드를 pulbic final로만 선언해도 불변 객체가 되지만, 이렇게 하면 다음 릴리스에서 내부 표현을 바꾸지 못하므로 권하지는 않음(아이템15,아이템16)

5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

  - 클래스에 가변 객체를 참조하는 필드가 하나라도 있다면 클라이언트에서 그 객체의 참조를 얻을 수 없도록 해야 함.

  - 이런 필드는 절대 클라이언트가 제공한 객체 참조를 가리키게 해서는 안 되며, 접근자 메서드가 그 필드를 그대로 받환해서도 안됨.

  - 생성자, 접근자, readObject 메서드(아이템 88) 모두에서 방어적 복사를 수행하라.

    ```java
    public final class Person{//2.클래스를 확장할 수 없도록 한다.
      private final Address address; //3.모든 필드를 final로 선언한다. // 4. 모든 필드를 final로 선언한다.
      
      public Person(Address address){
        this.address = address;
      }
      
      /* 5. 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다. */
      public Address getAddress(){
        /* 내부 가변 컴포넌트에 접근하게 한 경우 */
        //return address;
        /* 접근할 수 없도록 방어적 복사 */
        Address copyOfAddress = new Address();
        copyOfAddress.setStreet(address.getStreet());
        copyOfAddress.setAipCode(address.getZipCode());
        return new Address();
      }
      
      public static void main(String[] args){
        /* 내부 가변 컴포넌트에 접근하게 한 경우 */
        Adress seattle = new Address();
        seattle.setCity("Seattle");
        
        Person person = new Person();
        
        Address redmond = person.getAddress();
        redmond.setCity("Redmond");
        system.out.println(person.address.getCity());
        // 접근하게 한경우 : Redmond / 접근할 수 없도록 한 경우 : Seatle
      }
    }
    
    public class Address{
      private String zipCode;
      private String street;
      
      public String getZupCode(){
        return zipCode;
      }
      
      public setZipCode(String zipCode){
        this.zipCode = zipCode;
      }
      
      public String getStreet(){
        return street;
      }
      
      public setStreet(String street){
        this.street = street;
      }
    }
    ```

    > readObjtect
    >
    > - 매개변수로 바이트 스트림을 받는 생성자.

### 불변 클래스의 장점

- 함수형 프로그래밍에 적합하다.

  - 피연산자에 함수를 적용한 결과를 반환하지만 피연산자가 바뀌지는 않는다.

  > 함수형 프로그래밍 
  >
  > - 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 패턴

- 불변 객체는 단순하다.

  - 불변 객체는 생성된 시점의 상태를  파괴될 때까지 그대로 간직한다.
  - 모든 생성자가 클래스 불변식(class invariant)를 보장한다면 그 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다.

- 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다.

  - 여러 스레드가 동시에 사용해도 절대 훼손 되지 않는다.
  - 클래스를 스레드 안전하게 만드는 가장 쉬운 방법.

- 불변 객체는 안심하고 공유할 수 있다. 

  - 불변 객체에 대해서는 그 어떤 스레드도 다른 스레드에 영향을 줄 수 없으니 불변 객체는 안심하고 공유할 수 있다.
  - 불변 클래스라면 한번 만든 인스턴스를 최대한 재활용하기를 권한다.
    - 가장 쉬운 재활용 방법 : 자주 쓰이는 값들을 상수(public static final)로 제공하는 것.
  - 아무리 복사해봐야 원본과 똑같으니 복사 자체가 의미가 없다.
    - 불변 클래스는 clone 메서드나 복사 생성자(아이템13)를 제공하지 않는 게 좋다. 
    - String클래스의 복사 생성자는 이 사실을 잘 이해하지 못한 자바 초창기 때 만들어진 것으로, 되도록 사용하지 말아야 한다.(아이템 6)

- 불변 객체 끼리는 내부 데이터를 공유할 수 있다.

- 객체를 만들 때 불변 객체로 구성하면 이점이 많다.

  - 값이 바뀌지 않는 구성요소들로 이뤄진 객체라면 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하기 때문.
  - 불변 객체는 맵의 키와 집합(Set)의 원소로 쓰기에 안성맞춤.
    - 맵이나 집합은 안에 담긴 값이 바뀌면 불변식이 허물어지는데, 불변객체를 사용하면 그런 걱정은 하지않아도 됨.

- 실패 원자성을 제공한다.(아이템76)

  - 상태가 절대 변하지 않으니 잡깐이라도 불일치 상태에 빠질 가능성이 없다.

```java
public final class Complex{
  private final double re; //피연산자
  private final double im; //피연산자
  
  public Complex(double re, double im){
    this.re = re;
    this.im = im;
  }
  
  public double realPart(){return re;}
  public double imaginaryPart(){return im;}
  
  public Complex plus(Complex c){
    return new Complex(re + c.re, im + c.im);
  }
  
   public Complex minus(Complex c){
    return new Complex(re - c.re, im - c.im);
  }
}
```



### 불변 클래스의 단점

- 값이 다르다면 반드시 별도의 객체로 만들어야 한다.
  - "다단계 연산"을 제공하거나, "가변 동반 클래스"를 제공하여 대처할 수 있다.

### 불변 클래스를 만드는 다른 방법

- 상속을 막을 수 있는 또 다른 방법

  - private 또는 package-private 생성자 + 정적 팩터리

    ```java
    /*코드 17-2 생성자 대신 정적 팩터리를 사용한 불변 클래스*/
    public class Complex{
      private final double re;
      private final double im;
      
      private Complex(double re, double im){
        this.re = re;
        this.im = im;
      }
      
      public static Complex valueOf(double re, double im){
        return new Complex(re, im);
      }
      
      ... 
    }
    ```

  - 확장이 가능하다. 다수의 package-private 구현 클래스를 만들 수 있다.

  - 정적 팩터리를 통해 여러 구현 클래스 중 하나를 활용할 수 있는 유연성을 제공하고 객체 캐싱 기능으로 성능을 향상 시킬 수도 있다.

  

- 재 정의가 가능한 클래스는 방어적인 복사를 사용해야 한다.

- 모든 "외부에 공개하는" 필드가 final이어야 한다.

