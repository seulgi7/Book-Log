## 아이템 16. public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라.
### public 클래스에서 public 필드를 사용한 경우의 단점.
``` java
public class Point{
  public double x;
  public double y;
}
```
- 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.(아이템15)
  - API를 수정하지 않고는 내부 표현을 바꿀 수 없음.
  - 불변식을 보장할 수 없음.
  - 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없음.
 
### private 필드를 사용하고 접근자와 변경자(mutator) 메서드를 활용해 데이터를 캡슐화한다.
```java
Class Point{
  private double x;
  private double y;

  public Point(double x, double y){
    this.x = x;
    this.y = y;
  }

  public double getX() { return x; }
  public double getY() { return y; }

  public double setX() {this.x = x; }
  public double setY() {this.y = y; }
}
```
#### public 클래스라면? 이 방식이 확실히 맞음.
  - 이점 : 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있음.
  - 비교) public 클래스가 필드를 공개하면?ㅅ
    - 이를 사용하는 클라이언트가 생겨날 것이므로 내부 표현 방식을 마음대로 바꿀 수 없게 됨.
#### package-private 클래스 혹은 private 중첩 클래스라면? 데이터 필드를 노출한다 해도 하등의 문제가 없음.
``` java
class Point{
  /* package-private 클래스 (default) */
 // 멤버가 소속된 패키지 안의 모든 클래스에서 접근할 수 있음.
  public double x;
  public double y;
}
```
  - 그 클래스가 표현하려는 추상 개념만 올바르게 표현해주면 됨.
  - 클래스 선언 면에서나 이를 사용하는 클라이언트 코드 면에서나 접근자 방식보다 훨씬 깔끔.
    - package-private 클래스: 클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드 일뿐.
      → 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있음.
    - private 중첩 클래스: 수정범위가 더 좁아져서 이 클래스를 포함하는 외부 클래스까지로 제한됨.
      

#### public 클래스의 필드가 불변이라면? 직접 노출할 때의 단점이 조금은 줄어들지만, 결코 좋은생각 아님.
  - 단점은 여전
    - API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없음.
  - 이점 : 불변식은 보장할 수 있게 됨.
  ```java
    public final class Time{
      private static final int HOURS_PER_DAY = 24;
      private static final int MINUTES_PER_HOUR = 60;

      public final int hour;
      public final int minute;

      public Time(int hour, int minute){
        if(hour <0 || hour >= HOURS_PER_DAY)
          throw new IllegalArgumentException("시간: " + hour);
        if(minute <0 || minute >= MINUTES_PER_HOUR)
          throw new IllegalArgumentException("분: " + minute);
      }
      ... // 코드 생략
    }
  ```

> **핵심정리**
> public 클래스는 절대 가변 필드를 직접 노출해서는 안 됨.
> 불변 필드라면 노출해도 덜 위험하지만 완전히 안심할 수는 없음.
> package-private 클래스나 private 중첩 클래스에서는 종종 (불변이든 가변이든) 필드를 노출하는 편이 나을 때도 있음.
