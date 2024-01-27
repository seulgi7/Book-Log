# 아이템80. 스레드보다는 실행자, 테스크, 스트림을 애용하라.
큐
클라이언트가 요청한 작업을 백그라운드 스레드에 위임해 비동기적으로 처리해줌.
작업 큐가 필요 없어지면 클라이언트는 큐에 중단을 요청할 수 있고,그러면 큐는 남아 있는 작업을 마저 완료한 후 스스로 종료한다.


java.util.concurrent 패키지
- 실행자 프레임워크(Excutor Framework)라고 하는 인터페이스 기반의 유연한 테스크 실행 기능을 담고 있음.
- 작업 큐를 생성하는 법 : 단 한줄로 생성 가능.
  ```java
  ExecutorService exec = Executors.newSingleThreadExecutor();
  ```
- 생성한 실행자에 실행할 테스크(task:작업)을 넘기는 법.
  ```java
  exec.execute(runnable);
  ```
- 생성한 실행자를 종료시키는 방법
  ```java
  exec.shutdown();
  ```
- 실행자 서비스의 주요 기능들.
  - 특정 테스크가 완료되기를 기다린다.
    - ex.아이템 79의 예저 코드 79-2에서 본 get메서드
  - 테스크 모음 중 아무것 하나(invokeAny 메서드) 혹은 모든 테스크(invoke All 메서드)가 완료되기를 기다린다.
  - 실행자 서비스가 종료하기를 기다린다.
    - awaitTerminate 메서드.      
  - 완료된 테스크들의 결과를 차례로 받는다.
    - ExecutorCompleteService 이용.
  - 테스크를 특정 시간에 혹은 주기적으로 실행하게 한다.
    - ScheduledTread PoolExecutro 이용.      
  
