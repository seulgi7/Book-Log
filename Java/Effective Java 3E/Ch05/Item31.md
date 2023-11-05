# 아이템 31. 한정적 와일드카드를 사용해 API유연성을 높이라.

## 제네릭은 불공병

---

>  [공변과 불공변 ]
>
> - 공변  : A가 B의 하위 타입일때, T<A>가  T<B>의 하위 타입이면  T는 공변.
> - 불공변 : A가 B의 하위 타입일때, T<A>가  T<B>의 하위 타입이 아니면  T는 불공변.

- 매개변수화 타입은 불공변이다. (아이템28)
-  Type1과 Type2가 있을 때, List<Type1<Type1>>은 List<Type2<Type2>> 의 하위타입도 상위 타입도 아니다.
  - List<Object<Object>>에는 어떤 객체든 넣을 수 있지만 List<String<String>>에는 문자열만 넣을 수 있음.
  - 즉,  List<String<String>> 는 List<Object<Object>>가 하는 일을 제대로 수행하지 못하니 하위 타입이 될 수 없다.
- 때론 불공변 방식보다 유연한 방식이 필요할 때가 있다면? **한정적 와일드카드 타입**이라는특별한 매개변수화 타입이 유용하게 사용될 수 있음.



## 한정적 와일드카드 타입 - 생산자

-----

> [ 와일드카드 ]
>
> ```
> <?>
> ```
> - 정해지지 않은 unknown type이기 때문에  Collection<?>로 선언함으로써 모든 타입에 대해 호출이 가능해짐.

### 와일드 카드 타입을 사용하지 않은 pusthAll메서드

 일련의 원소를 Stack에 넣는 메서드를 추가해야한다고 가정해보자. 

```java
...
public void pushAll(Iterable<E> src){
  for(E e : src){
    push(e);
  }
}
...
```

- 이 메서드는 꺠끗이 컴파일되지만 완벽하진 않다.

- Iterable src원소 타입이 스택의 원소 타입과 일치하면 잘 작동한다.

- 하지만, 아래와 같이 `Number` 타입으로 선언된 Stack 객체의 메서드에 `Integer` 타입의 매개변수를 전달하면 컴파일 오류가 발생합니다. 

  ``` java
  ...
  Stack<Number> numberStack = new Stack<>();
  Iterable<Integer> integers = ...;
  numberStack.pushAll(integers);
  ...
  ```

  - 매개변수화 타입이 불공변이기때문에 `incompatible types... Iterable<Integer> cannot be converted to Iterable<Number>`와 같은 오류가 발생합니다.
  - 논리적으로는 Integer는 Number의 하위타입이므로 잘 동작해야 할 것 같지만, 에러가 발생한다.

- 해결책 : 자바는 이런 상황에 대처할 수 있는 **한정적 와일드카드 타입**이라는 특별한 매개변수화 타입을 지원한다.

### 생상자 매개변수에 와일드카드 타입 적용

```java
...
public void pushAll(Iterable<? extends E> src) {
    for (E e : src) {
        push(E);
    }
}
...
```

- 와일드카드 타입 **Iterable<? extends E>** 의미
  - pushAll의 입력 매개변수 타입은 E의 Iterable이 아니라 **E의 하위타입의 Iterable**이어야 한다.
- 이렇게 한정적 와일드카드를 사용하면 타입이 안전해지므로 클라이언트 코드가 정상적으로 컴파일된다.



## 한정적 와일드 카드 - 소비자(Cunsumer)

이번에는 Stack 인스턴스의 모든 원소를 매개변수로 받은 컬렉션으로 모두 옮기는 `popAll` 메서드를 작성해보자.

### 와일드카드 타입을 사용하지않은 pushAll메서드 

```java
public void popAll(Collection<E> dst) {
	while(!isEmpty()) {
  	dst.add(pop());
  }
}
```

- 주어진 컬렉션의 원소 타입이 스택의 원소 타입과 일치한다면 말끔히 컴파일되고 문제없이 동작함.
- stack<Number>의 원소를 Object용 컬렉션으로 옮기려 한다면? 타입이 일치하지않으므로 컴파일 에러가 발생.
- Number 클래스는 최상위 Object 클래스를 상속하지만 역시나 제네릭의 **매개변수화 타입은 불공변**이기 때문에 **상속이란 관계가 무의미**
- 해결책 : **와일드카드 타입**을 사용.

### 소비자(consumer) 매개변수에 와일드 카드 타입 적용

```java
// E의 상위 타입의 Collection이어야 한다.
public void popAll(Collection<? super E> dst) {
    while(!isEmpty()) {
        dst.add(pop());
    }
}
```

- 와일드카드 타입을 사용한 **Collection<? super E>** 의미
  - popAll의 입력 매개변수의 타입이 'E의 Collection'이 아니라 **E의 상위타입의 Collection**이어야 한다.
- stack<Number>의 원소를 Object용 컬렉션으로 옮기려 한다면? 타입이 일치하지않으므로 컴파일 에러가 발생하지 않음.
- **유연성을 극대화하려면 원소의 생산자나 소빙자용 입력 매개변수에 와일드 카드 타입을 사용하라.**

## PECS

와일드 카드 타입을 사용하는 기본 원칙.

> - Producer-Extends
>
> - Consumer-Super

****

- 매개변수화 타입T가 생산자(producer)라면 <? extends T>를 사용하자.
- 매개변수화 타입 T가 소비자(consumer)라면 <? super T>를 사용하자.

### PECS 공식 사용 전

```java
public static <E> Set<E> union(Set<E> s1, Set<E> s2)
```

- `s1`과 `s2`는 생산자이니 PECS 공식에 따라 생산자 와일드카드 방식에 따라 사용해야 한다.

### PECS 공식 사용 후

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
```

- 주의점) 반환타입에는 한정적 와일드 카드 타입을 사용하면 안된다. 유연성을 높여주기는 커녕 클라이언트 코드에서 와일드카드 타입을 사용해야하기 때문이다.

**[클라이언트 코드]**

```java
Set<Integer> integers = Set.of(1, 3, 5);
Set<Double> doubles = Set.of(2.0, 4.0, 6.0);
Set<Number> numbers = union(integers, doubles);
```

- 와일드카드가 제대로 사용된다면 사용자는 와일드 카드가 사용된지도 모른다. 만약 사용자가 와일드카드 타입을 신경써야 한다면 그 API는 문제가 있을 수 있다.
- 참고로, 만약 자바 버전이 7버전이라면 위 클라이언트 코드에서 명시적 타입 인수를 사용해주어야 한다. 자바 8이전까지는 타입 추론 능력이 충분하지 못해 반환타입을 명시해야 했다.




### 정리

유연성을 극대화하려면 원소의 생산자나 소비자용 입력 매개변수에 와일드카드 타입을 사용하자. 

생산자(producer)는 `extends`를 소비자(consumer)는 `super`를 사용한다. 

`Comparable`과 `Comparator`는 소비자라는 사실도 기억하자.
