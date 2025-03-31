# ExecutorService 우아한 종료
: 서버 기능을 업데이트를 위해서 서버를 재시작해야한다고 가정할 때, 서비스를 안정적으로 종료하는 것은 매우 중요하다.

## 1. ExecutorService의 종료 메서드

**서비스 종료**
- void shutdown()
   - 새로운 작업을 받지 않고, 이미 제출된 작업을 모두 완료한 후에 종료한다.
   - 논 블로킹 메서드(이 메서드를 호출한 스레드는 대기하지 않고 즉시 다음 코드를 호출한다.)
- List<Runnable> shutdownNow()
   - 실행 중인 작업을 중단하고, 대기 중인 작업을 반환하며 즉시 종료한다.
   - 실행 중인 작업을 중단하기 위해 인터럽트를 발생시킨다.
   - 논 블로킹 메서드

**서비스 상태 확인**
- boolean isShutdown()
  - 서비스가 종료되었는지 확인한다.
- boolean isTerminated()
  - shutdown(), shutdownNow() 호출 후, 모든 작업이 완료되었는지 확인한다.
 
**작업 완료 대기**
- boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException
   - 서비스 종료시 모든 작업이 완료될 때까지 대기한다. 이때 지정된 시간까지만 대기한다.
   - 블로킹 메서드
 
**close()**
- shutdown()을 호출하고, 하루를 기다려도 작업이 완료되지 않으면 shutdownNow()를 호출한다.
- 호출한 스레드에 인터럽트가 발생해서 shutdownNow()를 호출한다.

## 1-1. shutdown() - 처리 중인 작업이 없는 경우
- ExecutorService에 아무런 작업이 없고, 스레드만 2개 대기하고 있다.
- shutdown()을 호출한다.
- ExecutorService는 새로운 요청을 거절한다.
- 스레드 풀의 자원을 정리한다.

## 1-2. shutdown() - 처리 중인 작업이 있는 경우
- shutdown()을 호출한다.
- ExecutorService는 새로운 요청을 거절한다.
- 스레드 풀의 스레드는 처리중인 작업을 완료한다.
- 스레드 풀의 스레드는 큐에 남아있는 작업도 모두 꺼내서 완료한다.
- 모든 작업을 완료하면 자원을 정리한다.
- 결과적으로 처리중이던 taskA, taskB는 물론이고, 큐에 대기중이던 taskC, taskD도 완료된다.

## 1-3. shutdownNow() - 처리중인 작업이 있는 경우
- shutdownNow()를 호출한다.
- ExecutorService는 새로운 요청을 거절한다.
- 큐를 비우면서, 큐에 있는 작업을 모두 꺼내서 컬렉션으로 반환한다.
  - List<Runnable> runnables = es.shutdownNow()
- 작업 중인 스레드에 인터럽트가 발생한다.
   - 작업 중인 taskA, taskB는 인터럽트가 걸린다.
   - 큐에 대기 중인 taskC, taskD는 수행되지 않는다.
- 작업을 완료하면 자원을 정리한다.

  ## 2. ExecutorService 우아한 종료 - 구현
  ```
  public class ExecutorShutdownMain {

    public static void main(String[] args) throws InterruptedException {
        ExecutorService es = Executors.newFixedThreadPool(2);
        es.execute(new RunnableTask("taskA"));
        es.execute(new RunnableTask("taskB"));
        es.execute(new RunnableTask("taskC"));
        es.execute(new RunnableTask("longTask", 100_000)); // 100초 대기
        shutdownAndAwaitTermination(es);
  
      }
    static void shutdownAndAwaitTermination (ExecutorService es) {
        es.shutdown(); // non-blocking , 새로운 작업을 받지 않는다. 처리 중이거나, 큐에 이미 대기중인 작업은 처리한다. 이후에 풀의 스레드를 종료한다.

        try {

            if(10초 후에도 작업이 완료되지 않으면) {
                es.shutdownNow(); //작업 취소

                if(10초후에도 작업이 취소되지 않으면) {
                }
            }
         } catch (InterruptedException ex) {
            es.shutdownNow();
          }
      
    }
  
  }
  ```
- **서비스 종료**
  ```
  es.shutdown();
  ```
- 새로운 작업을 받지 않는다. 처리 중이거나, 큐에 이미 대기 중인 작업은 처리한다. 이후에 풀의 스레드를 종료한다.
- shutdown()은 블로킹 메서드가 아니다. 서비스가 종료될 때까지 main 스레드가 대기하지 않는다. main 스레드는 바로 다음 코드를 호출한다.

```
if(!es.awaitTermination(10, TimeUnit.SECONDS)) {
}
```
- 블로킹 메서드이다.
- main 스레드는 대기하며 서비스 종료를 10초간 기다린다.
   - 만약 10초 안에 모든 작업이 완료된다면 true를 반환한다.
   - 그런데 longTask는 10초가 지나도 완료되지 않았다.
   - 따라서 false를 반환한다.
 
- **서비스 정상 종료 실패 -> 강제 종료 시도 **
  ```
  es.shutdownNow();

  if(!es.awaitTermination(10, TimeUnit.SECONDS)) {
  }
  ```
- 정상 종료가 10초 이상 너무 오래 걸렸다.
- shutdownNow()를 통해 강제 종료에 들어간다. shutdown()과 마찬가지로 블로킹 메서드가 아니다.
**- 강제 종료를 하면 작업 중인 스레드에 인터럽트가 발생한다. 다음 로그를 통해 인터럽트를 확인할 수 있다.**
- 인터럽트가 발생하면서 스레드도 작업을 종료하고, shutdownNow()를 통한 강제 shutdown도 완료된다.
- 
