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

### 1-1. shutdown() - 처리 중인 작업이 없는 경우
- ExecutorService에 아무런 작업이 없고, 스레드만 2개 대기하고 있다.
- shutdown()을 호출한다.
- ExecutorService는 새로운 요청을 거절한다.
- 스레드 풀의 자원을 정리한다.

### 1-2. shutdown() - 처리 중인 작업이 있는 경우
- shutdown()을 호출한다.
- ExecutorService는 새로운 요청을 거절한다.
- 스레드 풀의 스레드는 처리중인 작업을 완료한다.
- 스레드 풀의 스레드는 큐에 남아있는 작업도 모두 꺼내서 완료한다.
- 모든 작업을 완료하면 자원을 정리한다.
- 결과적으로 처리중이던 taskA, taskB는 물론이고, 큐에 대기중이던 taskC, taskD도 완료된다.

### 1-3. shutdownNow() - 처리중인 작업이 있는 경우
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
 
- **서비스 정상 종료 실패 -> 강제 종료 시도**
  ```
  es.shutdownNow();

  if(!es.awaitTermination(10, TimeUnit.SECONDS)) {
  }
  ```
- 정상 종료가 10초 이상 너무 오래 걸렸다.
- shutdownNow()를 통해 강제 종료에 들어간다. shutdown()과 마찬가지로 블로킹 메서드가 아니다.
**- 강제 종료를 하면 작업 중인 스레드에 인터럽트가 발생한다. 다음 로그를 통해 인터럽트를 확인할 수 있다.**
- 인터럽트가 발생하면서 스레드도 작업을 종료하고, shutdownNow()를 통한 강제 shutdown도 완료된다.

- **서비스 종료 실패**
- 마지막에 강제종료인 es.shutdownNow()를 호출출한 다음에 왜 10초간 또 기다릴까?
- shutdownNow()가 작업 중인 스레드에 인터럽트를 호출하는 것은 맞다.
- 인터럽트 이후에, 자원을 정리하는 어떤 간단한 작업을 수행할 수도 있다.
- 이런 시간을 기다려 주는 것이다.
- 극단적이지만 최악의 경우 스레드가 다음과 같이 인터럽트를 받을 수 없는 코드를 수행중일 수 있다.
  ```
  while(true) {}
  ```
- 이 경우 인터럽트 예외가 발생하지 않고, 스레드도 계속 수행된다.
- 이런 스레드는 자바를 강제 종료해야 제거할 수 있다.
 
## 3. Executor 스레드 풀 관리 - 코드
```
public class PoolSizeMainV1 {

   public static void main(String[] args) throws InterruptedException {

      BlockingQueue<Runnable> 작업을 보관할 큐 = new ArrayBlockingQueue<>(2);
      ExecutorService es = new ThreadPoolExecutor(기본 스레드 수 , 최대 스레드 수 , 초과 스레드가 생존할 수 있는 대기시간, 단위, 작업을 보관할 큐);
      printState(es);

      es.execute(new RunnableTask("task1");
      es.execute(new RunnableTask("task2");
      es.execute(new RunnableTask("task3");
      es.execute(new RunnableTask("task4");
      es.execute(new RunnableTask("task5");
      es.execute(new RunnableTask("task6");

      try {
         es.execute(new RunnableTask("task7"));
      } catch (RejectedExecutionException e) {
   
      }
      es.close();
   }
}
```

```
BlockingQueue<Runnable> workQueue = new ArrayBlockingQueue<>(2);
ExecutorService es = new ThreadPoolExecutor(2,4,3000, TimeUnit.MILLISECONDS, workQueue);
```
### 3-1. Executor 스레드 풀 관리 - 분석
1. 작업을 요청하면 core 사이즈만큼 스레드를 만든다.
2. core 사이즈를 초과하면 큐에 작업을 넣는다.
3. 큐를 초과하면 max 사이즈만큼 스레드를 만든다. 임시로 사용되는 초과 스레드가 생성된다.
   - 큐가 가득차서 큐에 넣을 수도 없다. 초과 스레드가 바로 수행해야 한다.
4. max 사이즈를 초과하면 요청을 거절한다. 예외가 발생한다.
    - 큐도 가득차고, 풀에 최대 생성 가능한 스레드 수도 가득 찼다. 작업을 받을 수 없다.
  
## 4. Executor 전략
### 4-1. 고정 풀 전략 
**newFixedThreadPool(nThreads)**
- 스레드 풀에 nThread 만큼의 기본 스레드를 생성한다. 초과 스레드는 생성하지 않는다.
- 큐 사이즈에 제한이 없다.(LinkedBlockingQueue)
- 스레드 수가 고정되어 있기 때문에 CPU, 메모리 리소스가 어느 정도 예측 가능한 안정적인 방식이다.
```
 new ThreadPoolExecutor(nThreads, nThreads, 0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>())
```

```
public class PoolSizeMainV2 {

   public static void main(String[] args) throws InterruptedException {

      ExecutorService es = Executors.newFixedThreadPool(2);

      for (int i = 1; i<=6 ; i++ ) {
         String taskName = "task" + i;
         es.execute(new RunnableTask(taskName));
      }
      es.close();
   }
}
```

**특징**
- 스레드 수가 고정되어 있기 때문에 CPU, 메모리 리소스가 어느 정도 예측 가능한 안정적인 방식이다.
- 큐 사이즈도 제한이 없어서 작업을 많이 담아두어도 문제가 없다.

**문제**
- 사용자가 확대되거나 갑작스런 요청이 증가할 때는 장점이 단점이 되기도 한다.
- 고정 스레드 전략은 실행되는 스레드 수가 고정되어 있다. 따라서 사용자가 늘어나도 CPU, 메모리 사용량이 확 늘어나지 않는다.
- 요청이 처리되는 시간보다 쌓이는 시간이 더 빠르면, 문제가 될 수 있다.
- 결국 서버 자원은 여유가 있는데, 사용자만 점점 느려지는 문제가 발생한 것이다.

### 4-2. 캐시 풀 전략
**newCachedThreadPool()**
- 기본 스레드를 사용하지 않고, 60초 생존 주기를 가진 초과 스레드만 사용한다.
- 초과 스레드의 수는 제한이 없다.
- 큐에 작업을 저장하지 않는다.(SynchronousQueue)
  - 대신에 생산자의 요청을 스레드 풀의 소비자 스레드가 직접 받아서 바로 처리한다.
- 모든 요청이 대기하지 않고 스레드가 바로바로 처리한다. 따라서 빠른 처리가 가능하다.

```
new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<Runnable>());
```
SynchronousQueue는 아주 특별한 블로킹 큐이다.
- BlockingQueue 인터페이스의 구현체 중 하나이다.
- 이 큐는 내부에 저장 공간이 없다. 대신에 생산자의 작업을 소비자 스레드에게 직접 전달한다.
- 쉽게 이야기해서 저장 공간의 크기가 0이고, 생산자 스레드가 작업을 전달하면 소비자 스레드가 큐에서 작업을 꺼낼 때까지 대기한다.
- 소비자 작업을 요청하면 기다리던 생산자가 소비자에게 직접 작업을 전달하고 반환된다. 그 반대의 경우도 같다.
- 이름 그대로 생산자와 소비자를 동기화하는 큐이다.

```
public class PoolSizeMainV3 {

   public static void main (String[] args) throws InterruptedException {

      ThreadPoolExecutor es = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 3, TimeUnit.SECONDS, new SynchronousQueue<>());

      for (int i=1; i<=4; i++) {
         String taskName = "task" + i;
         es.execute(new RunnableTask(taskName));
      }
      es.close();
   }
}
```

**특징**
- 캐시 스레드 풀 전략은 매우 빠르고, 유연한 전략이다.
- 이 전략은 기본 스레드도 없고, 대기 큐에 작업도 쌓이지 않는다.
- 대신에 작업 요청이 오면 초과 스레드로 작업을 바로바로 처리한다.
- 초과 스레드의 수도 제한이 없기 때문에 CPU, 메모리 자원만 허용한다면 시스템의 자원을 최대로 사용할 수 있다.
- 초과 스레드의 유지 시간이 있기 때문에 요청이 갑자기 증가하면 스레드도 갑자기 증가하고, 요청이 줄어들면 스레드도 점점 줄어든다.
- 작업의 요청 수에 따라서 스레드도 증가하고 감소한다.

**문제**
- 갑작스런 요청이 증가하면 너무 많은 스레드가 작업을 처리하면서 시스템 전체가 느려지는 현상이 발생한다.
- 캐시 스레드 풀 전략은 스레드가 무한으로 생성될 수 있다.

- 
- 수 천개의 스레드가 처리하는 속도보다 더 많은 작업이 들어온다.
- 시스템은 너무 많은 스레드에 잠식당해서 거의 다운된다. 메모리도 거의 다 사용되어 버린다.
- 시스템이 멈추는 장애가 발생한다. 

### 4-3. 사용자 정의 풀 전략
  
