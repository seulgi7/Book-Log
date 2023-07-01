## 아이템1. 생성자 대신 정적 팩터리 메서드를 고려하라.

클래스는 생성자와 별도로 정적 팩터리 메소드(static factory method)를 제공할 수 있다.


### 정적 팩터리 메서드가 생성자보다 좋은 장점 

1. **이름을 가질 수 있다.**

   - 정적 팩터리 
     - 생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다.
       - 생성자 : 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.
       - ex ) 값이 소수인 BigInteger를 반환한다 : 생성자 BigInteger(int, int, Random)  **VS** BigInteger.probablePrime()
     - 한 클래스에 시그니처가 같은 생성자가 여러 개 필요할 것 같으면, 생성자를 정적 팩터리 메서드로 바꾸고 각가의 차이를 잘 드러내는 이름을 지을 수 있다.
       - 생성자:하나의 시그니처로는 생성자를 하나만 만들 수 있다.

2. **호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.**

   - 불변 클래스(Immutable class) : 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용하는 식으로 불필요한 객체 생성을 피할 수 있음.

     - ex) Boolean.valueOf(boolean) 메서드 : 객체를 아예 생성하지 않음 --> 성능 UP!

   - 인스턴스 통제(instance-controlled)클래스 : 반복되는 요청에 같은 객체를 반환하는 식으로 정적 팩터리 방식의 클래스는 언제 어는 인스턴스를 살아 있게 할 지를 철저히 통제할 수 있다.

     - [플라이웨이트 패턴[Gamma95]](https://refactoring.guru/ko/design-patterns/flyweight)의 근간이 되며, 열거 타입은 인스턴스가 하나만 만들어짐을 보장함.
     - 인스턴스를 통제하는 이유 ?
       - 클래스를 싱클턴으로 만들 수 있음.
       - 인스턴스화 불가(noninstanitable)로 만들 수 있음.
       - 불변 값클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있음.

     ```java
     package me.whiteship.chapter01.item01;
     
     /**
      * 이 클래스의 인스턴스는 #getInstance()를 통해 사용한다.
      * @see #getInstance()
      */
     public class Settings {
     
         private boolean useAutoSteering;
     
         private boolean useABS;
     
         private Difficulty difficulty;
     
         private Settings() {}
     
         private static final Settings SETTINGS = new Settings();
     
         public static Settings getInstance() {
             return SETTINGS;
         }
     
     }
     ```

     ```java
     package me.whiteship.chapter01.item01;
     
     import java.util.EnumSet;
     import java.util.Set;
     
     public class Product {
     
         public static void main(String[] args) {
             Settings settings1 = Settings.getInstance();
             Settings settings2 = Settings.getInstance();
     
             System.out.println(settings1);
             System.out.println(settings2);
     
             Boolean.valueOf(false);
             EnumSet.allOf(Difficulty.class);
         }
     }
     ```

     

3. **반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.**
   - 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 '엄청난 유연성'을 선물한다.
   - 인터페이스를 정적 팩터리 메서드의 반환 타입으로 사용하는 인터페이스 기반 프레임워크(아이템 20)을 만드는 핵심 기술이기도 함.

4. **입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.**
   - ex) EnumSet 클래스(아이템 36) : public 생성자 없이 오직 정적 팩토리만 제공하는데, OpenJDK에서는 원소의 수에 따라 두 가지 하위 클래스 중 하나의 인스턴스를 반환함.
     - 원소가 64개 이하  → RegularEnumSet의 인스턴스 반환 ; 원소들을 long 변수 하나로 관리.
     - 원소가 65개 이상  → JumboEnumSet의 인스턴스 반환; 원소들을 long 배열로 관리.

5. **정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.**



### 정적팩터리 메서드가 생성자보다 좋지않은 단점

1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.
2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

