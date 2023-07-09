# 3장. 모든객체의 공통 메서드

- 이번 장 목표 :  final이 아닌 Object메서드들을 언제 어떻게 재정의 해야하는가 ?

  - Object : 객체를 만들 수 있는 구체 클래스이지만 기본적으로 상속해서 사용하도록 설계됨.

    - Object에서 final이 아닌 메서드(equls, hashCode, toString, clone, finalize)는 모두 재정의(overriding)을 염두에 두고 설계된 것.

      → 재정의 시 지켜야 하는 일반 규약이 명확이 정의되어 있음.

      → Object를 상속하는 클래스는 이 메서드들을 일반 규약에 맞게 재정의해야 한다.

## 아이템10. equals는 일반 규약을 지켜 재정의하라.

- equals를 재정의 하지 않는 것이 최선인 경우.

  (다음에서 열거한 상황 중 하나에 해당한다면 재정의하지 않는 것이 최선.)

  - 각 인스턴스가 본질적으로 고유하다.
    - 값을 표현하는 게 아니라 동작하는 개체를 표현하는 클래스가 여기 해당.
    - 예: 싱글톤, Enum, 쓰레드 등 

  - 인스턴스의 '논리적 동치성(logical equality)'를 검사할 일이 없다.

    - 기본적인 equls : 객체의 동일성을 비교.
    - 논리적인 동치성의 예
      - 값

  - 상위 클래스에서 재정의한 equlas가 하위 클래스에도 딱 들어맞는다.

    - 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아 쓰고, List구현체들은 AbstarctList로부터, Map 구현체들은 AbstractMap으로부터 상속받아 그대로 쓴다.

  - 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.

    

- equals를 재정의해야 할 때

  - 객체 식별성(object identity;두 객체가 물리적으로 같은가)이 아니라 논리적 동치성을 확인해야 하는데, 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의되지 않았을 때.

    - 주로 값 클래스들이 여기에 해당함.

      > 값 클래스란?
      >
      >  Integer와 String 처럼 값을 표현하는 클래스를 말한다.

      - 두 값 객체를 equals로 비교하는 프로그래머는 객체가 같은지가 아니라 값이 같은지를 알고 싶어 할 것.

    - equals가 논리적 동치성을 확인하도록 재정의해두면, 그 인스턴스는 값을 비교하길 원하는 프로그래머의 기대에 부응함은 물론 Map의 키와 Set의 원소로 사용할 수 있게 된다.

    - 값 클래스라도 재정의 하지 않아도 될 때

      - 인스턴스 통제 클래스(아이템 1)

      - Enum(아이템 34)

        - 어차피 논리적으로 같은 인스턴스가 2개 이상 만들어지지 않으니 논리적 동치성과 객체 식별성이 사실상 똑같은 의미가 된다.

          → Object의 equals가 논리적 동치성까지 확인해준다고 볼 수 있다.

          

- equals 메서드를 재정의해야 할 때 반드시 따라야 하는 일반 규약

  - 반사성(reflexivity)

    - null이 아닌 모든 참조 값 x에 대해, x.equls(x)는 true다.
    - 객체는 자기 자신과 같아야 한다는 뜻.

  - 대칭성(symmetry)

    - null이 아닌 모든 참조 값 x, y에 대해, x.equals(x)가 true면 y.equals(x)도 true다.

    - 두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻.

    - 잘못된 코드

      ```java
      // 코드 10-1 잘못된 코드 - 대칭성 위배! (54-55쪽)
      public final class CaseInsensitiveString {
          private final String s;
      
          public CaseInsensitiveString(String s) {
              this.s = Objects.requireNonNull(s);//널(Null) 체크를 위한 메소
          }
      
      		// 대칭성 위배!
          @Override 
        	public boolean equals(Object o) {
              if (o instanceof CaseInsensitiveString)
                	//equalsIgnoreCase : 대소문자 구분없이 문자열 자체만으로 비교
                  return s.equalsIgnoreCase(
                          ((CaseInsensitiveString) o).s);
              if (o instanceof String)  
                  return s.equalsIgnoreCase((String) o);// --> 한 방향으로만 작동한다!
              return false;
          }
      
          // 문제 시연 (55쪽)
          public static void main(String[] args) {
              CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
              String s = "polish";
              System.out.println(cis.equals(s)); //true
              System.out.println(s.equals(cis)); //false
          }
      
      }
      ```

      - 문제는 CaseInsensitive String의 equals는 일반 String을 알고 있지만 String의 equals는 CaseInsensitiveString의 존재를 모른다는 데 있다.

        → s.equals(cis)는 false를 반환하여, 대칭성을 명백히 위반함!!

    - 올바른 코드

      ```java
      public final class CaseInsensitiveString {
          private final String s;
      
          public CaseInsensitiveString(String s) {
              this.s = Objects.requireNonNull(s);
          }
      
      	
          @Override 
        	public boolean equals(Object o) {
              if (o instanceof CaseInsensitiveString)
                  return s.equalsIgnoreCase(
                          ((CaseInsensitiveString) o).s);//true
          }
      
          // 문제 시연 (56쪽)
          public static void main(String[] args) {
              CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
              CaseInsensitiveString cis2 = new CaseInsensitiveString("polish");
              System.out.println(cis.equals(polish)); //true
              System.out.println(cis2.equals(cis)); //true
          }
      
      }
      ```

      

  - 추이성(transitivity)

    - null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true이고 y.equals(z)도 true면 x.equals(z)도 true다.

    - 첫 번째 객체와 두 번째 객체가 같고, 두번째 객체와 세 번째 객체가 같다면, 첫 번째 객체와 세 번째 객체도 같아야 한다.

    - 구체 클래스를 확장해 새로운 값을 추가하면서 equals 규약을 만족시킬 방법은 존재하지 않는다.

      ```java
      public class ColorPoint extends Point {
          private final Color color;
      
          public ColorPoint(int x, int y, Color color) {
              super(x, y);
              this.color = color;
          }
      
          // 코드 10-3 잘못된 코드 - 추이성 위배! (57쪽)
          @Override public boolean equals(Object o) {
              if (!(o instanceof Point))
                  return false;
      
              // o가 일반 Point면 색상을 무시하고 비교한다.
              if (!(o instanceof ColorPoint))
                  return o.equals(this);
      
              // o가 ColorPoint면 색상까지 비교한다.
              return super.equals(o) && ((ColorPoint) o).color == color;
          }
      
          public static void main(String[] args) {
              // equals 메서드(코드 10-3)는 추이성을 위배한다. (57쪽)
              ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
              Point p2 = new Point(1, 2);
              ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
              System.out.printf("%s %s %s%n",
                                p1.equals(p2), p2.equals(p3), p1.equals(p3)); //true true false
          }
      }
      
      ```

      ```JAVA
      // 단순한 불변 2차원 정수 점(point) 클래스 (56쪽)
      public class Point {
      
          private final int x;
          private final int y;
      
          public Point(int x, int y) {
              this.x = x;
              this.y = y;
          }
      
          @Override public boolean equals(Object o) {
              if (this == o) {
                  return true;
              }
      
              if (!(o instanceof Point)) {
                  return false;
              }
      
              Point p = (Point) o;
              return p.x == x && p.y == y;
          }
      }
      ```

      

    - 구체 클래스의 하위 클래스에서 값을 추가할 방법은 없지만 괜찮은 우회 방법 존재 : "상속 대신 컴포지션을 활용해라." (아이템18)

      ```java
      // 코드 10-5 equals 규약을 지키면서 값 추가하기 (60쪽)
      public class ColorPoint {
          private final Point point;
          private final Color color;
      
          public ColorPoint(int x, int y, Color color) {
              point = new Point(x, y);
              this.color = Objects.requireNonNull(color);
          }
      
          /**
           * 이 ColorPoint의 Point 뷰를 반환한다.
           */
          public Point asPoint() {
              return point;
          }
      
          @Override public boolean equals(Object o) {
              if (!(o instanceof ColorPoint))
                  return false;
              ColorPoint cp = (ColorPoint) o;
              return cp.point.equals(point) && cp.color.equals(color);
          }
      }
      ```

      - Point를 상속하는 대신 Point를 ColorPoint의 private 필드로 두고, ColorPoint와 같은 위치의 일반 Point를 반환하는 뷰(view)메서드(아이템63)을 public으로 추가하는 식.

      

  - 일관성(consistency)

    - null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true를 반환하거나 항상 false를 반환한다.
    - 두 객체가 같다면(어느 하나 혹은 두 객체 모두가 수정되지 않는 한) 앞으로도 영원히 같아야 한다는 뜻.

  - null-아님

    - null이 아닌 모든 참조 값 x에 대해, x.equls(null)은 false다.
    - 모든 객체가 null과 같지 않아야 한다.

  

- equals 메소드 구현 방법

  1. == 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
     - float와 double을 제외한 기본 타입 필드는 == 연산자로 비교.
     - float double 필드는 각각 정적 메서드인 Floate.compare(float, float), Double.compare(double, double)로 비교.
     - 참조 타입 필드는 각각의 equals로 메소드로 비교.
  2. instanceof 연산자로 입력이 올바른 타입인지 확인한다.
  3. 입력을 올바른 타입으로 형변환한다.
  4. 입력 객체와 자기 자신의 대응되는 '핵심'필드 들이 모두 일치하는지 하나씩 검사한다.

  ```java
  public class Point {
  
      private final int x;
      private final int y;
  
      public Point(int x, int y) {
          this.x = x;
          this.y = y;
      }
  
      @Override public boolean equals(Object o) {
          if (this == o) {
              return true;
          }
  
          if (!(o instanceof Point)) {
              return false;
          }
  
          Point p = (Point) o;
          return p.x == x && p.y == y;
      }
  }
  
  ```



- 주의사항
  - equals를 재정의할 땐 hashCode도 반드시 재정의하자.(아이템 11)
  - 너무 복잡하게 해결하려 들지 말자.
  - Object 외의 타입을 매개변수로 받는 equals 메서드는 선언하지 말자.

> **핵심정리**
>
> 꼭 필요한 경우가 아니면 equals를 재정의하지 말자.
>
> 많은 경우에 Object의 equals가 여러분이 원하는 비교를 정확히 수행해준다.
>
> 재정의가 필요할 때는 그 클래스의 핵심  필드를 모두 빠짐없이, 다섯 가지 규약을 확실히 지켜가며 비교해야 한다.
