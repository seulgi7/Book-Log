문자열 연결 연산자(+)
- 여러 문자열을 하나로 합쳐주는 편리한 수단
- 그러나 한 줄짜리 출력값 혹은 작고 크기가 고정된 객체의 문자열 포현을 만들 때라면 괜찮지만, 본격적으로 사용하기 시작하면 성능 저하를 감내하기 어렵다.
- 문자열 연결 연산자로 문자열 n개를 잇는 시간: n2에 비례한다.

- 성능을 포기하고 싶지 않다면?
  -  String 대신 StringBuilder의 append를 사용하자. 
  -  StringBuilder는 문자열 개수에 따라 선형으로 증가한다.

```
// 문자열 연결 사용
public String statement() {
	String result = "";
    for (int i = 0; i < numItems(); i++)
    	result += lineForItem(i);
	return result;
}

// StringBuilder 사용
public String statement2() {
	StringBuilder b = new StringBuilder(numItems() * LINEWIDTH);
    for (int i = 0; i < numItems(); i++)
    	b.append(lineForItem(i));
	return b.toString();
}
```

statement와 statement2의 수행시간을 비교?
- 저자 컴퓨터 기준 statement2가 6.5배나 빨랐다고 한다.

