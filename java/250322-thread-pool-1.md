# 스레드 풀과 Executor 프레임워크

## 1. 스레드를 직접 사용할 때의 문제점
- 1. 스레드 생성 비용으로 인한 성능 문제
- 2. 스레드 관리 문제
- 3. Runnable 인터페이스의 불편함

### 1-1. 스레드 생성 비용으로 인한 성능 문제
스레드를 사용하려면 먼저 스레드를 생성해야 한다. 그런데 스레드는 다음과 같은 이유로 매우 무겁다.
- 메모리 할당 : 각 스레드는 자신만의 호출 스택(call stack)을 가지고 있어야 한다. 이 호출 스택은 스레드가 실행되는 동안 사용하는 메모리 공간이다.
- 운영체제 자원 사용 : 스레드를 생성하는 작업은 운영체제 커널 수준에서 이루어지며, 시스템 콜(system call)을 통해 처리된다.
- 운영체제 스케줄러 설정 : 새로운 스레드가 생성되면 운영체제의 스케줄러는 이 스레드를 관리하고 실행 순서를 조정해야 한다.
이런 문제를 해결하려면 생성한 스레드를 재사용하는 방법을 고려할 수 있다. 스레드를 재사용하면 처음 생성할 때를 제외하고는 생성을 위한 시간이 들지 않는다.

### 1-2. 스레드 관리 문제
- 서버의 CPU, 메모리 자원은 한정되어 있기 때문에, 스레드는 무한하게 만들 수 없다.
- 예를 들어, 사용자의 주문을 처리하는 서비스라고 가정한다.
- 평소 동시에 100개 정도의 스레드면 충분했는데, 갑자기 10000개의 스레드가 필요한 상황이 된다면, CPU, 메모리 자원이 버티지 못할 것이다.
- 또한, 다음과 같은 문제도 있다.
- 애플리케이션을 종료해야 할 때, 안전한 종료를 위해 실행 중인 스레드가 남은 작업은 모두 수행한 다음에 프로그램을 종료해야 하는 상황이라 가정한다.
- 이런 경우에도 스레드가 어딘가에 관리가 되어 있어야 한다.

### 1-3. Runnable 인터페이스의 불편함
```
public interface Runnable {
  void run();
}
```
Runnable 인터페이스는 다음과 같은 이유로 불편하다.
- **반환 값이 없다** : run() 메서드는 반환 값을 가지지 않는다. 따라서 실행 결과를 얻기 위해서는 별도의 매커니즘을 사용해야 한다.
쉽게 이야기해서 스레드의 실행 결과를 직접 받을 수 없다. 
스레드가 실행한 결과를 멤버 변수에 넣어두고, join() 등을 사용해서 스레드가 종료되길 기다린 다음에 멤버 변수에 보관한 값을 받아야 한다.
- **예외 처리** : run() 메서드는 체크 예외(checked exception)를 던질 수 없다. 체크 예외의 처리는 메서드 내부에서 처리해야 한다.
이런 문제를 해결하려면 반환 값도 받을 수 있고, 해당 스레드에서 발생한 예외도 받을 수 있는 방법이 필요하다.

### 1-4. 해결
- 지금까지 설명한 1-1, 1-2 문제를 해결하려면 스레드를 생성하고 관리하는 풀(Pool)이 필요하다.
- 스레드를 관리하는 스레드 풀에 스레드를 미리 필요한 만큼 만들어둔다.
  - 1. 스레드는 스레드 풀에서 대기하며 쉰다.
  - 2. 작업 요청이 온다.
  - 3. 스레드 풀에서 이미 만들어진 스레드를 하나 조회한다.
  - 4. 조회한 스레드1로 작업을 처리한다.
  - 5. 스레드1은 작업을 완료한다.
  - 6. 작업을 완료한 스레드는 종료하는게 아니라, 다시 스레드 풀에 반납한다. 스레드1은 이후에 다시 재사용될 수 있다.

## 2. Executor 프레임워크 소개
### 2-1. Executor 프레임워크의 주요 구성 요소
```
public interface Executor {
  void execute(Runnable command);
}
```
- 가장 단순한 작업 실행 인터페이스로, execute(Runnable command) 메서드 하나를 가지고 있다.
  
```
public interface ExecutorService extends Executor, AutoCloseable {
    <T> Future<T> submit(Callable<T> task);

    @Override
    default void clse(){...}
}
```
- Executor 인터페이스를 확장해서 작업 제출과 제어 기능을 추가로 제공한다.
- Executor 프레임워크를 사용할 때는 대부분 이 인터페이스를 사용한다.
- ExecutorService 인터페이스의 기본 구현체는 ThreadPoolExecutor이다.

### 2-2. 시작
```
public class RunnableTask implements Runnable {
    private final String name;
    private int sleepMs = 1000;

    public RunnableTask(String name) {
      this.name = name;
    }

    @Override
    public void run() {

      sleep(sleepMs);
    }
}
```

- Runnable 인터페이스를 구현한다. 1초의 작업이 걸리는 간단한 작업이다.

```public class ExecutorBasicMain {
      public static void main(String[] args) throws InterruptedException {

        ExecutorService es = new ThreadPoolExecutor(2,2,0,TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());
        es.execute(new RunnableTask("taskA"));
        es.execute(new RunnableTask("taskB"));
        es.execute(new RunnableTask("taskC"));
        es.execute(new RunnableTask("taskD"));

        printState(es);

        sleep(3000);

        es.close();

        printState(es);
      }
  }
```
- ExecutorService의 가장 대표적인 구현체는 ThreadPoolExecutor이다.
- ThreadPoolExecutor는 크게 2가지 요소로 구성되어 있다.
  - 스레드 풀 : 스레드를 관리한다.
  - BlockingQueue : 작업을 보관한다. 생산자 소비자 문제를 해결하기 위해 단순한 큐가 아니라, BlockingQueue를 사용한다.
  - 생산자가 es.execute(new RunnableTask("taskA"))를 호출하면, RunnableTask("taskA") 인스턴스가 BlockingQueue에 보관된다.
    - **생산자** : es.execute(작업)를 호출하면 내부에서 BlockingQueue에 작업을 보관한다. main 스레드가 생산자가 된다.
    - **소비자** : 스레드 풀에 있는 스레드가 소비자이다. 이후에 소비자 중에 하나가 BlockingQueue에 들어있는 작업을 받아서 처리한다.
   
**ThreadPoolExecutor 생성자**
ThreadPoolExecutor의 생성자는 다음 속성을 사용한다.
- corePoolSize:  스레드 풀에서 관리되는 기본 스레드 수
- maximumPoolSize : 스레드 풀에서 관리되는 최대 스레드 수
- keepAliveTime, TimeUnit unit : 기본 스레드 수를 초과해서 만들어진 스레드가 생존할 수 있는 대기 시간. 이 시간동안 처리할 작업이 없다면 초과 스레드는 제거된다.
- BlockingQueue workQueue : 작업을 보관할 블로킹 큐

### 2-3. 실행 결과 분석
- ThreadPoolExecutor를 생성한 시점에 스레드 풀에 스레드를 미리 만들어두지는 않는다.
- main 스레드가 es.execute("taskA ~ taskD")를 호출한다.
  - 참고로, main 스레드는 작업을 큐에 보관까지만 하고 바로 다음 코드를 수행한다.
- task A~D 요청이 블로킹 큐에 들어온다.
- 최초의 작업이 들어오면 이때 작업을 처리하기 위해 스레드를 만든다.
   - 참고로 스레드 풀에 스레드를 미리 만들어두지는 않는다.
- 작업이 들어올 때마다 corePoolSize의 크기까지 스레드를 만든다.
   - 예를 들어서 최초 작업인 taskA가 들어오는 시점에 스레드1을 생성하고, 다음 작업인 taskB가 들어오는 시점에 스레드2를 생성한다.
   - 이런 방식으로, corePoolSize에 지정한 수만큼 스레드를 스레드 풀에 만든다. 여기서는 2를 설정했으므로 2개까지 만든다.
   - corePoolSize까지 스레드가 생성되고 나면, 이후에는 스레드를 생성하지 않고 앞서 만든 스레드를 재사용한다.
   - 참고로, 스레드 풀의 스레드가 작업을 실행할 때, 스레드의 상태가 변경된다.
- 작업이 완료되면 스레드 풀에 스레드를 반납한다. 스레드를 반납하면 스레드는 대기(WAITING) 상태로 스레드 풀에 대기한다.
   - 참고로 실제 반납하는 게 아니라, 스레드의 상태가 변경된다.
- 반납된 스레드는 재사용된다.
- taskC, taskD의 작업을 처리하기 위해, 스레드 풀에 있는 스레드를 재사용한다.
- 작업이 완료되면 스레드는 다시 풀에서 대기한다.
- close()를 호출하면 ThreadPoolExecutor가 종료된다. 이때 스레드 풀에 대기하는 스레드도 함께 제거된다.

## 3. Runnable의 불편함
### 3-1. Runnable 사용
- Runnable 인터페이스는 다음과 같은 불편함이 있다.
 - **반환 값이 없다.** : 스레드의 실행 결과를 직접 받을 수 없다. 스레드가 실행한 결과를 멤버 변수에 넣어두고, join()등을 사용해서 스레드가 종료되길 기다린 다음에 멤버 변수를 통해 값을 받아야 한다.
 - **예외 처리** : run() 메서드는 체크 예외를 던질 수 없다. 체크 예외의 처리는 메서드 내부에서 처리해야 한다.

### 3-2. 예시 
```
public calss RunnableMain {
      public static void main(String[] args) throws InterruptedException {
        MyRunnable task = new MyRunnable();
        Thread thread = new Thread(task, "Thread-1");
        thread.start();
        thread.join();
        int result = task.실행 결과를 담아두는 변수;
    }

    static class MyRunnable implements Runnable {
        int 실행 결과를 담아두는 변수;

        @Override
        public void run() {
          실행 결과를 담아두는 변수 = new Random().nextInt(10);
        }
    }
}
```

### 3-3. 실행 결과 분석
- 프로그램이 시작되면 Thread-1이라는 별도의 스레드를 하나 만든다.
- Thread-1이 수행하는 MyRunnable은 무작위 값을 하나 구한 다음에 value 필드에 보관한다.
- 클라이언트인 main 스레드가 이 별도의 스레드에서 만든 값을 얻어오려면 Thread-1 스레드가 종료될 때까지 기다려야 한다.
- 그래서 main 스레드는 join()을 호출해서 대기한다.
- 이후에 main 스레드에서 MyRunnable 인스턴스의 value 필드를 통해 최종 무작위 값을 획득한다.

### 3-4. 정리
- 별도의 스레드에서 만든 값 하나를 받아오는 과정은 다음과 같다.
- 작업 스레드(Thread-1)는 값을 어딘가에 보관해두어야 하고, 요청 스레드(main)는 작업 스레드의 작업이 끝날 때까지 join()을 호출해서 대기한 다음에, 어딘가에 보관된 값을 찾아서 꺼내야 한다.
- 만약, 작업 스레드는 간단히 값을 return을 통해 반환하고, 요청 스레드는 그 반환 값을 바로 받을 수 있다면 코드가 훨씬 간결해질 것이다.
- 이런 문제를 해결하기 위해 Executor 프레임워크는 Callable 과 Future라는 인터페이스를 도입했다.

## 4. Future
### 4-1. Runnable 과 Callable 비교
```
public interface Runnable {
    void run();
}
```
- Runnable의 run()은 반환 타입이 void 이다. 따라서 값을 반환할 수 없다.
- 예외가 선언되어 있지 않다. 따라서 해당 인터페이스를 구현하는 모든 메서드는 체크 예외를 던질 수 없다.
  - 참고로 자식은 부모의 예외 범위를 넘어설 수 없다. 부모에 예외가 선언되어 있지 않으므로 예외를 던질 수 없다.
  - 물론 런타임(비체크) 예외는 제외다.
 
```
public interface Callable<V> {
    V call() throws Exception;
}
```
- Callable의 call()은 반환 타입이 제네릭 V이다. 따라서 값을 반환할 수 있다.
- throws Exception 예외가 선언되어 있다. 따라서 해당 인터페이스를 구현하는 모든 메서드는 체크 예외인 Exception과 그 하위 예외를 모두 던질 수 있다.

### 4-2. 예제
public class CallableMainV1 {
    public static void main(String [] args) throws ExecutionException, InterruptedException {
      ExectorService es = Executors.newFixedThreadPool(1);
      Future<Integer> future = es.submit(new MyCallable());
      Integer result = future.get();
      es.close();
    }

    static class MyCallable implements Callable<Integer> {
        @Override
        public Integer call() {
            int value = new Random().nextInt(10);
            return value;
        }
    }
} 
- java.util.concurrent.Executors 가 제공하는 newFixedThreadPool(size)을 사용하면 편리하게 ExecutorService를 생성할 수 있다.
- 아래 두 코드는 동일한 코드이다.
```
 ExecutorService es = new ThreadPoolExecutor(1,1,0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());              
```

```
 ExecutorService es = Executors.newFixedThreadPool(1);
```
- Mycallable을 구현하는 부분을 보면, 숫자를 반환하므로 반환할 제네릭 타입을 <Integer>로 선언했다.
- Runnable과의 차이는, 결과를 필드에 담아두지 않고 결과를 반환한다는 점이다. 따라서 결과를 보관할 별도의 필드를 만들지 않아도 된다.

**submit()**
```
<T> Future<T> submit(Callable<T> task); // 인터페이스 정의
```
ExecutorService가 제공하는 submit()을 통해 Callable을 작업으로 전달할 수 있다.

```
Future<Integer> future = es.submit(new MyCallable());
```
MyCallable 인스턴스가 블로킹 큐에 전달되고, 스레드 풀의 스레드 중 하나가 이 작업을 실행할 것이다.
이때 작업의 처리 결과는 직접 반환되는 것이 아니라 Future라는 특별한 인터페이스를 통해 반환된다.

```
Integer result = future.get();
```
future.get()을 호출하면 MyCallable의 call()이 반환한 결과를 받을 수 있다.

**Executor 프레임워크의 강점**
- 스레드를 생성하거나 join()으로 스레드를 제어하는 코드가 필요 없다.
- 단순하게 ExecutorService에 필요한 작업을 요청하고 결과를 받아서 쓰면 된다.

**그러나, 요청 스레드가 future.get()을 호출했을 때, 스레드 풀의 스레드가 아직 작업을 처리중이라면 어떻게 될까?**
**또한, 왜 결과를 바로 반환하지 않고, 불편하게 Future라는 객체를 대신 반환할까?**

### 4-3. 분석
- Future는 번역하면 미래라는 뜻이고, 여기서는 미래의 결과를 받을 수 있는 객체라는 뜻이다.
- 그렇다면 누구의 미래의 결과를 말하는 것일까?
  ```
  Future<Integer> future = es.submit(new MyCallable());
  ```
- submit()의 호출로 MyCallable의 인스턴스를 전달한다.
- 이때 sumbit()은 MyCallable.call()이 반환하는 숫자 대신에 Future를 반환한다.
- 생각해보면 MyCallable이 즉시 실행되어서 즉시 결과를 반환하는 것은 불가능하다.
- 왜냐하면, 스레드 풀의 스레드가 미래의 어떤 시점에 이 코드를 대신 실행해야 한다.
- MyCallable.call() 메서드는 호출 스레드가 실행하는 것도 아니고, 스레드 풀의 다른 스레드가 실행하기 때문에 언제 실행이 완료되어서 결과를 반환할 지 알 수 없다.
- 따라서 es.submit()은 MyCallable의 결과를 반환하는 대신에 MyCallable의 결과를 나중에 받을 수 있는 Future라는 객체를 대신 제공한다.
- 정리하면 Future는 객체를 통해 전달한 작업의 미래 결과를 받을 수 있다.

### 4-4. 예제 코드 분석
```
Future<Integer> future =  es.submit(new MyCallable());
```
- submit()을 호출해서 ExecutorService에 taskA를 전달한다.
  **Future의 생성**
- ExecutorService는 전달한 taskA의 미래 결과를 알 수 있는 Future 객체를 생성한다.
   - Future는 인터페이스이다. 이때 생성되는 실제 구현체는 FutureTask이다.
- 그리고 생성한 Future 객체 안에 taskA 의 인스턴스를 보관한다.
- Future는 내부에 taskA 작업의 완료 여부와, 작업의 결과 값을 가진다.
- submit()을 호출한 경우 Future가 만들어지고, 전달한 작업인 taskA가 바로 블로킹 큐에 담기는 것이 아니라, taskA를 감싸고 있는 Future가 대신 블로킹 큐에 담긴다.
- 작업을 전달할 때 생성된 Future는 즉시 반환되기 때문에, 요청 스레드는 대기하지 않고 자유롭게 본인의 다음 코드를 호출할 수 있다.

**스레드1**
- 큐에 들어있는 Future[taskA]를 꺼내서 스레드 풀의 스레드1이 작업을 시작한다.
- 참고로 Future의 구현체인 FutureTask는 Runnable 인터페이스도 함께 구현하고 있다.
- 스레드1은 FutureTask의 run() 메서드를 수행한다.
- 그리고 run() 메서드가 taskA의 call() 메서드를 수행하고 그 결과를 받아서 처리한다.

```
Integer result = future.get();
```

**요청 스레드**
- 요청 스레드는 Future 인스턴스의 참조를 가지고 있다.
- 요청 스레드는 future.get()을 호출하고, 아직 작업이 완료되지 않았으므로 Future 가 완료 상태가 될 때까지 대기한다.
- 이때 요청 스레드의 상태는 RUNNABLE -> WAITING 이 된다.

```future.get()```을 호출했을 때
- **Future가 완료 상태** : Future 가 완료 상태면 Future에 결과도 포함되어 있다. 이 경우 요청 스레드는 대기하지 않고, 값을 즉시 반환받을 수 있다.
- **Future가 완료 상태가 아님** : taskA가 아직 수행되지 않았거나 수행중이다. 이때는 요청 스레드가 결과를 받기 위해 대기해야 한다.
                                  요청 스레드가 마치 락을 얻을 때처럼, 결과를 얻기 위해 대기한다. 이처럼 스레드가 어떤 결과를 얻기 위해 대기하는 것을 블로킹(Blocking)이라 한다.

**요청 스레드**
- 대기(WAITING) 상태로 future.get()을 호출하고 대기중이다.

**스레드1**
- taskA 작업을 완료한다.
- Future에 taskA의 반환 결과를 담는다.
- Future의 상태를 완료로 변경한다.
- 요청 스레드를 깨운다. 요청 스레드는 WAITING -> RUNNABLE 상태로 변한다.

**요청 스레드**
- 요청 스레드는 RUNNABLE 상태가 되었다. 그리고 완료 상태의 Future에서 결과를 반환받는다.

**스레드1**
- 작업을 마친 스레드1은 스레드 풀로 반환된다. RUNNABLE -> WAITING

### 4-5. Future가 필요한 이유
#### 4-5-1. Future를 반환하는 코드
```
Future<Integer> future1 = es.submit(task1);
Future<Integer> future2 = es.submit(task2);

Integer sum1 = future1.get();
Integer sum2 = future2.get();
```

- ExecutorService는 Future를 반환한다. 
- 요청 스레드는 task1을 ExecutorService에 요청한다.
  - 요청 스레드는 즉시 Future를 반환 받는다.
  - 작업 스레드1은 task1을 수행한다.
- 요청 스레드는 taks2를 ExecutorService에 요청한다.
   - 과정은 task1에서와 동일하다.

요청 스레드는 task1, task2를 동시에 요청할 수 있다. 따라서 두 작업은 동시에 수행된다.
- 이후에 요청 스레드는 future1.get()을 호출하며 대기한다.
  - 작업 스레드1이 작업을 진행하는 약 2초간 대기하고 결과를 받는다.
- 이후에 요청 스레드는 future2.get()을 호출하며 즉시 결과를 받는다.
  - 작업 스레드는 이미 2초간 작업을 완료했다. 따라서 future2.get()은 거의 즉시 결과를 반환한다.

#### 4-5-2. Future 없이 결과를 직접 반환하는 코드(가정)
```
Integer sum1 = es.submit(task1);
Integer sum2 = es.submit(task2);
```

- 만약, ExecutorService가 Future 없이 결과를 직접 반환한다고 가정해본다.
- 요청 스레드는 task1을 ExecutorService에 요청하고 결과를 기다린다.
  - 작업 스레드가 작업을 수행하는데 2초가 걸린다.
  - 요청 스레드는 결과를 받을 때까지 2초간 대기한다.
  - 요청 스레드는 2초 후에 결과를 받고 다음 라인을 수행한다.
- 요청 스레드는 task2를 ExecutorService에 요청하고 결과를 기다린다.
  - 과정은 task1에서와 동일한다.

 Future를 사용하지 않는 경우 결과적으로 task1의 결과를 기다린 다음에 task2를 요청한다.
 따라서 총 4초의 시간이 걸렸다. 이것은 마치 단일 스레드가 작업을 한 것과 비슷한 결과이다.

 #### 4-5-3. 정리
 - Future라는 개념이 없다면 결과를 받을 때까지 요청 스레드는 아무일도 못하고 대기해야 한다. 따라서 다른 작업을 동시에 수행할 수도 없다.
 - Future라는 개념 덕분에 요청 스레드는 대기하지 않고, 다른 작업을 수행할 수 있다.
 - Future는 요청 스레드를 블로킹(대기) 상태로 만들지 않고, 필요한 요청을 모두 수행할 수 있게 해준다.
 - 필요한 모든 요청을 한 다음에 Future.get()을 통해 블로킹 상태로 대기하며 결과를 받으면 된다.

## 5. Future - 취소
```
public class FutureCancelMain {
    private static boolean 실행중인 작업을 인터럽트시킬지 여부 = true;

    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(1);
        Future<String> future = es.submit(new MyTask());

        boolean cancelResult1 = future.cancel(실행중인 작업을 인터럽트시킬지 여부);

        try {
            future.get();
        } catch(CancellationException e) {

        } catch(InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        es.close();
    }

    static class MyTask implements Callable<String> {

      @Override
      public String call() {
          try {
              for(int i=0<;i<10;i++){
                작업 중;
              }
          } catch (InterruptedException e) {
              return "Interrupted";
          }
          return "Completed";
      }
    } 
 }
```

- cancel(true)를 호출했다.
- 실행 중인 작업에 인터럽트가 발생해서 실행 중인 작업을 중지 시도한다.
- 이후 Future.get()을 호출하면 CancellationException 런타임 예외가 발생한다.

- 만약, cancel(false)를 호출한다면 실행중인 작업은 그냥 둔다.
- 실행중인 작업은 그냥 두더라도 cancel()을 호출했기 때문에 Future는 CANCEL 상태가 된다.
- 이후 Future.get()을 호출하면 CancellationException 런타임 예외가 발생한다.

## 6. Future - 예외
```
public class FutureExceptionMain {
    public static void main(String[] args) {
      ExecutorService es = Executors.newFixedThreadPool(1);
      Future<Integer> future = es.submit(new ExCallable());

    try {
      Integer result = future.get();
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    } catch (ExecutionException e) {
        Throwable cause = e.getCause();
    }
    es.close();
    }

    static class ExCallable implements Callable<Integer> {
      @Override
      public Integer call() {
        throw new IllegalStateException("ex!");
      }
    }
}
```
- **요청 스레드** : es.submit(new ExCallable())을 호출해서 작업을 전달한다.
- **작업 스레드** : ExCallable을 실행하는데, IllegalStateException 예외가 발생한다.
   - 작업 스레드는 Future에 발생한 예외를 담아둔다. 참고로 예외도 객체이다.
   - 예외가 발생했으므로 Future 의 상태는 FAILED가 된다.
- **요청 스레드** : 결과를 얻기 위해 future.get()을 호출한다.
   - Future의 상태가 FAILED면 IllegalStateException 예외를 던진다.
   - 이 예외는 내부에 앞서 Future에 저장해둔 IllegalStateException을 포함하고 있다.
   - e.getCause()을 호출하면 작업에서 발생한 원본 예외를 받을 수 있다.
 
## 7. ExecutorService - 작업 컬렉션 처리
- ExecutorService는 여러 작업을 한 번에 편리하게 처리하는 invokeAll(), invokeAny() 기능을 제공한다.
**invokeAll()**
  - 모든 Callable 작업을 제출하고 모든 작업이 완료될 때까지 기다린다.
  - 매개변수로 timeout, unit을 전달하면 지정된 시간동안 기다린다.
**invokeAny()**
  - 하나의 Callable 작업이 완료될 때까지 기다리고, 가장 먼저 완료된 작업의 결과를 반환한다.
  - 완료되지 않은 나머지 작업은 취소한다.
  - 매개변수로 timeout, unit을 전달하면 지정된 시간동안 기다린다.
 
```
public class InvokeAllMain {
    public static void main(String[] args) throws  ExecutionException,InterruptedException {
      ExecutorService es = Executors.newFixedThreadPool(10)

       CallableTask task1 = new CallableTask("task1", 1000);
       CallableTask task2 = new CallableTask("task2", 2000);
       CallableTask task3 = new CallableTask("task3", 3000);
       List<CallableTask> tasks = List.of(task1, task2, task3);

       List<Future<Integer>> futures = es.invokeAll(tasks);
         for (Future<Integer> future : futures) {
             Integer value = future.get();
        }
        es.close()
    }
}
```
- `invokeAll()`은 한번에 여러 작업을 제출하고, 모든 작업이 완료될 때까지 기다린다.

```
public class InvokeAnyMain {
   public static void main(String[] args) throws  ExecutionException,InterruptedException {
      ExecutorService es = Executors.newFixedThreadPool(10)

       CallableTask task1 = new CallableTask("task1", 1000);
       CallableTask task2 = new CallableTask("task2", 2000);
       CallableTask task3 = new CallableTask("task3", 3000);
       List<CallableTask> tasks = List.of(task1, task2, task3);

       Integer value = es.invokeAny(tasks);
       es.close();
    }
  
}
```
- `invokeAny()`는 한 번에 여러 작업을 제출하고, 가장 먼저 완료된 작업의 결과를 반환한다.
- 이때 완료되지 않은 나머지 작업은 인터럽트를 통해 취소한다.

## 8. ExecutorService 주요 메서드 정리
**작업 제출 및 실행**
- void execute(Runnable command) : Runnable 작업을 제출한다. 반환값이 없다.
- <T> Future<T> submit(Callable<T> task) : Callable 작업을 제출하고 결과를 반환받는다.
- Future<?> submit(Runnable task) : Runnable 작업을 제출하고 결과를 반환 받는다. Runnable은 반환값이 없기 때문에 future.get()을 호출할 경우 null 을 반환한다.
