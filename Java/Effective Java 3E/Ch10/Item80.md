# 아이템80. 스레드보다는 실행자, 테스크, 스트림을 애용하라.
- 과거에는 클라이언트가 요청한 작업을 백그라운드 스레드에 위임해 비동기적으로 처리하기 위해 단순한 작업 큐를 사용했다.
-  하지만 이는 안전 실패나 응답 불가에 대한 예외 코드 작성 등으로 수많은 코드 작업이 발생했다.
- 다행히도 실행자 프레임워크의 등장으로 이제 이러한 수고를 덜 수 있게 되었다.
## 실행자 프레임워크(Executor Framework)
- 실행자 프레임워크는 java.util.concurrent 패키지에 속해있으며, 인터페이스 기반의 유연한 태스크 실행 기능을 담고 있다.
- 장점 : 기존보다 모든 면에서 뛰어난 작업 큐를 이제는 단 한 줄로 생성할 수 있다.
```java
// 작업 큐 생성
ExecutorService exec = Executors.newSingleThreadExecutor();

// 실행할 태스크를 넘김
exec.execute(runnable);

// 실행자 종료
exec.shutdown();
  ```
## 실행자 서비스의 주요 기능
1. 특정 태스크가 완료되기를 기다림.
     - get() 메서드를 통해 태스크가 완료되기를 기다릴 수 있다.
``` java
ExecutorService exec = Executors.newSingleThreadExecutor();

exec.submit(()  -> s.removeObserver(this)).get();
```
2. 태스크 모음 중 아무것 하나(invokeAny 메서드) 혹은 모든 태스크(invokeAll 메서드)가 완료되기를 기다림.
```java
ExecutorService exec = Executors.newSingleThreadExecutor();

exec.submit(()  -> s.removeObserver(this)).get();
```
3. 실행자 서비스가 종료하기를 기다림(awaitTermination 메서드).
``` java
exec.awaitTermination(10, TimeUnit.SECONDS);
```
4. 완료된 태스크들의 결과를 차례로 받는다(ExecutorCompletionService 이용).
``` java
ExecutorService exex = Executors.newFixedThreadPool(2);
ExecutorCompletionService executorCompletionService = 
    new ExecutorCompletionService(exex);

.....


for (int i = 0; i < 2; i++) {
    executorCompletionService.take().get();
}
```
5. 태스크를 특정 시간에 혹은 주기적으로 실행하게 한다(ScheduledThreadPoolExecutor 이용).
``` java
ScheduledThreadPoolExecutor scheduledExecutor =
                new ScheduledThreadPoolExecutor(1);
``` 
## 스레드 풀
 - 스레드 풀의 스레드 개수는 고정할 수도 있고 필요에 따라 늘어나거나 줄어들게 설정할 수 있습니다.
 - 우리가 필요한 실행자 대부분은 java.util.concurrent.Executors의 정적 팩토리들을 이용해 생성할 수 있습니다.
 
 - 평범하지 않은 실행자를 원한다면 ThreadPoolExecutor 클래스를 직접 사용하면 됩니다.
   - 이 클래스로는 스레드 풀 동작을 결정하는 거의 모든 속성을 제어할 수 있습니다.
 
## 실행자 서비스를 사용하기 까다로운 경우 : Executors.newCachedThreadPool
 - 일반적으로 작은 프로그램이나 가벼운 서버에 적합합니다.
 - CachedThreadPool에서는 요청받은 태스크들이 큐에 쌓이지 않고 즉시 스레드에 위임돼 실행됩니다. 가용한 스레드가 없다면 새로 하나를 생성합니다.
 
 - 서버가 아주 무겁다면 CPU 이용률이 100%로 치닫고, 새로운 태스크가 도착하는 족족 또 다른 스레드를 생성하며 상황을 더욱 악화시킬 수 있습니다.
 
-  따라서 무거운 프로덕션 서버에서는 스레드 개수를 고정한 Executors.newFixedThreadPool을 선택하거나, 완전히 통제할 수 있는 ThreadPoolExecutor를 직접 사용하는 편이 낫습니다.
## 스레드를 직접 다루는 것을 삼가자
- 작업 큐를 손수 만드는 일과 스레드를 직접 다루는 일은 삼가야 한다.
- 스레드를 직접 다루면 스레드가 작업 단위와 수행 매커니즘 역할을 모두 수행하지만, 실행자 프레임워크를 사용하면 작업 단위와 실행 매커니즘이 분리된다.
- 작업 단위를 나타내는 핵심 추상 개념은 태스크이고, 이 태스크는 Runnable과 Callable로 나눌 수 있다.(Callable은 Runnable과 비슷하지만 값을 반환하고 임의의 예외를 던질 수 있다.)
- 태스크를 수행하는 일반적인 매커니즘이 바로 실행자 서비스다.
    - 태스크 수행을 실행자 서비스에 맡기면 원하는 태스크 수행 정책을 선택할 수 있고, 언제든 변경할 수 있다.
    - 이 말인 즉, 실행자 프레임워크가 작업 수행을 담당해준다는 것이다.
 
### 예 :  포크-조인(fork-join)
 - 자바7부터 실행자 프레임워크는 포크-조인(fork-join) 태스크를 지원하도록 확장되었습니다.
 - 포크-조인 태스크는 포크-조인 풀이라는 특별한 실행자 서비스가 실행해줍니다.
 - ForkJoinTask의 인스턴스는 작은 하위 태스크로 나뉠 수 있고, ForkJoinPool을 구성하는 스레드들이 이 태스크들을 처리하며, 일을 먼저 끝낸 스레드는 다른 스레드의 남은 태스크를 가져와 대신 처리할 수도 있습니다.
