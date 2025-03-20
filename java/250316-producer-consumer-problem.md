# 1. 생산자 소비자 문제
- 멀티스레드 프로그래밍에서 자주 등장하는 동시성 문제 중 하나로, 여러 스레드가 동시에 데이터를 생산하고 소비하는 상황을 다룬다.

## 1-1. 기본 개념
- 생산자 : 데이터를 생성하는 역할을 한다. 예를 들어, 파일에서 데이터를 읽어오거나 네트워크에서 데이터를 받아오는 스레드가 생산자 역할을 할 수 있다.
- 소비자 : 생성된 데이터를 사용하는 역할을 한다. 예를 들어, 데이터를 처리하거나 저장하는 스레드가 소비자 역할을 할 수 있다.
- 버퍼 : 생산자가 생성한 데이터를 일시적으로 저장하는 공간이다. 이 버퍼는 한정된 크기를 가지며, 생산자와 소비자가 이 버퍼를 통해 데이터를 주고받는다.

## 1-2. 문제 상황
- 생산자가 너무 빠를 때
: 버퍼가 가득 차서 더 이상 데이터를 넣을 수 없을 때까지 생산자가 데이터를 생성한다. 버퍼가 가득 찬 경우 생산자는 버퍼에 빈 공간이 생길 때까지 기다려야 한다.
- 소비자가 너무 빠를 때
: 버퍼가 비어서 더 이상 소비할 데이터가 없을 때까지 소비자가 데이터를 처리한다. 버퍼가 비어있을 때 소비자는 버퍼에 새로운 데이터가 들어올 때까지 기다려야 한다

## 2. 예제 1

## 2-1. 버퍼
```
public interface BoundedQueue {
  void put(String data);

  String take();
}
```
- BoundedQueue : 버퍼 역할을 하는 큐의 인터페이스이다.
- put(data) : 버퍼에 데이터를 보관한다. (생산자 스레드가 호출하고, 데이터를 생산한다.)
- take() : 버퍼에 보관된 값을 가져간다. (소비자 스레드가 호출하고, 데이터를 소비한다.)

## 2-2. 버퍼 구현체
```
public class BoundedQueueV1 implements BoundedQueue {
  private final 데이터를 보관하는 버퍼 = new ArrayDeque<>();
  private final int 버퍼에 저장할 수 있는 최대 크기;

  public BoundedQueueV1(int max) {
      this.max = max;
  }

  @Override
  public synchronized void put(String data) {
    if(큐가 가득차면) {
        return;
    }
    큐에 데이터를 저장;
  }

  @Override
  public synchronized String take() {
    if(큐가 비어있으면) {
      return null;
    }
    return 큐에서 꺼낸 데이터;
  }
}
```
** 임계 영역**
- 여기서 핵심 공유 자원은 바로 데이터를 보관하는 버퍼이다.
- 여러 스레드가 접근할 예정이므로 synchronized를 사용해서 한 번에 하나의 스레드만 put() 또는 take()를 실행할 수 있도록 안전한 임계 영역을 만든다.

## 2-3. 생산자 스레드
```
public class ProducerTask implements Runnable {
    private BoundedQueue 버퍼;
    private String 전달된 데이터;

    @Override
    public void run() {
      큐에 데이터를 저장;
    }
}
```
## 2-4. 소비자 스레드
```
public class ConsumerTask implements Runnable {
    private BoundedQueue 버퍼;

    @Override
    public void run() {
      큐에서 꺼낸 데이터;
    }
}
```
## 2-5. main 클래스
```
public class BoundedMain {
    public static void main(String[] args) {
      BoundedQueue 버퍼 = new BoundedQueueV1(버퍼에 저장할 수 있는 최대 크기);

      생산자 먼저 실행
      //소비자 먼저 실행
    }

    private static void 생산자 먼저 실행(BoundedQueue 버퍼) {
      List<Thread> threads = new 스레드를 담을 배열<>();
      생산자 스레드 실행(버퍼, 스레드를 담을 배열);
      소비자 스레드 실행(버퍼, 스레드를 담을 배열);
    }

    private static void 소비자 먼저 실행(BoundedQueue 버퍼) {
      List<Thread> threads = new 스레드를 담을 배열<>();
      소비자 스레드 실행(버퍼, 스레드를 담을 배열);
      생산자 스레드 실행(버퍼, 스레드를 담을 배열);
    }

    private static void startProducer(BoundedQueue 버퍼, List<Thread> 스레드를 담을 배열) {
      for(){
          Thread producer = new Thread(new ProducerTask(queue, "이름"), "이름");
          스레드를 담을 배열.add(producer);
          producer.start();
        }
    }

    private static void startProducer(BoundedQueue 버퍼, List<Thread> 스레드를 담을 배열) {
      for(){
          Thread consumer = new Thread(new ConsumerTask(queue, "이름"), "이름");
          스레드를 담을 배열.add(consumer);
          consumer.start();
        }
    }
}
```

## 2-6. 결과 분석
- 생산자 스레드를 전부 실행하고, 이후에 소비자 스레드를 실행한다.
- 버퍼에 저장할 수 있는 최대 크기를 넘어서면, 더는 큐에 데이터를 추가할 수 없다.
- 데이터를 버리지 않으려면, 생산자 스레드가 기다리는 것이다.
- 언젠가는 소비자 스레드가 실행되어서 큐의 데이터를 가져갈 것이고, 큐에 빈 공간이 생기게 된다. 이때 큐의 데이터를 보관하는 것이다.

- 소비자 스레드를 먼저 실행하는 경우에는, 데이터가 없기 때문에 데이터를 소비할 수 없다.
- 언젠가 생산자 스레드가 실행되어서 큐에 데이터를 추가할 때까지 기다리는 것이 대안이다.
- 물론 생산자 스레드가 계속해서 데이터를 생산한다는 가정이 필요하다.
 

# 3. 예제 2
## 3-1. 버퍼 구현체
```
public class BoundedQueueV2 implements BoundedQueue {
  private final 데이터를 보관하는 버퍼 = new ArrayDeque<>();
  private final int 버퍼에 저장할 수 있는 최대 크기;

  public BoundedQueueV1(int max) {
      this.max = max;
  }

  @Override
  public synchronized void put(String data) {
    while(큐가 가득차면) {
        sleep();
    }
    큐에 데이터를 저장;
  }

  @Override
  public synchronized String take() {
    while(큐가 비어있으면) {
        sleep();
    }
    return 큐에서 꺼낸 데이터;
  }
}
```

- `put()` : 큐에 빈 공간이 생기는지 주기적으로 체크한다. 만약 빈 공간이 없다면 `sleep()`을 사용해서 잠시 대기하고, 깨어난 다음에 다시 반복문에서 큐의 빈 공간을 체크한다.
- `take()` : 큐에 데이터가 있는지 주기적으로 체크한다. 만약 데이터가 없다면 `sleep()`을 사용해서 잠시 대기하고, 깨어난 다음에 다시 반복문에서 큐에 데이터가 있는지 체크한다.

## 3-2. 결과 분석
- 생산자 스레드 p3 는 임계 영역에 들어가기 위해 먼저 락을 획득한다.
- 큐에 데이터를 저장하려고 시도한다. 그런데 큐가 가득 차있다.
- sleep(1000)을 사용해서 잠시 대기한다. 이때 RUNNABLE -> TIMED_WAITING 상태가 된다.
- 이때 반복문을 사용해서 1초마다 큐에 빈 자리가 있는지 반복해서 확인한다.
   - 빈 자리가 있다면 큐에 데이터를 입력하고 완료된다.
   - 빈 자리가 없다면 sleep()으로 잠시 대기한 다음 반복문을 계속해서 수행한다.
  
**여기서 핵심은, p3 스레드가 락을 가지고 있는 상태에서, 큐에 빈 자리가 나올 때까지 대기한다는 점이다..**

## 3-3.무한 대기 문제
- 소비자 스레드 c1가 임계 영역에 들어가기 위해 락을 획득하려 한다.
- 그런데 락이 없다! 왜냐하면 p3 가 락을 가지고 임계 영역에 이미 들어가 있기 때문이다.
- p3가 락을 반납하기 전까지는 c1은 절대로 임계 영역에 들어갈 수 없다.
- 여기서 문제가 발생한다.
   - p3가 락을 반납하려면 -> 소비자 스레드인 c1이 먼저 작동해서 큐의 데이터를 가져가야 한다.
   - 소비자 스레드인 c1이 락을 획득하려면 -> 생산자 스레드인 p3가 먼저 락을 반납해야 한다.
- p3는 락을 반납하지 않고, c1은 큐의 데이터를 가져갈 수 없다.
- 지금 상태면 p3는 절대로 락을 반납할 수 없다.
- 왜냐하면 락을 반납하려면 c1이 먼저 큐의 데이터를 소비해야 한다. 그래야 p3가 큐에 데이터를를 저장하고 임계 영역을 빠져나가며 락을 반납할 수 있다.
- 그런데 p3가 락을 가지고 임계 영역 안에 있기 때문에, 임계 영역 밖의 c1은 락을 획득할 수 없으므로, 큐에 접근하지 못하고 무한 대기한다.
- 결과적으로 소비자 스레드인 c1은 p3가 락을 반납할 때까지 BLOCKED 상태로 대기한다.
- 나머지 소비자 스레드 c2, c3도 BLOCKED 상태로 대기한다.

## 3-3. 정리
- 버퍼가 비었을 때 소비하거나, 버퍼가 가득 찼을 때 생산하는 문제를 해결하기 위해, 단순히 스레드가 잠깐 기다리면 될 것이라 생각했는데, 문제가 더 심각해졌다.
- 결국 임계 영역 안에서 락을 가지고 대기하는 것이 문제이다.
- 그래서 다른 스레드가 임계 영역 안에 접근조차 할 수 없는 것이다.
- 잘 생각해보면, 락을 가지고 임계 영역 안에 있는 스레드가 sleep()을 호출해서 잠시 대기할 때는 다른 스레드에게 락을 양보하면 어떨까?
- 그러면 다른 스레드가 버퍼에 값을 채우거나 버퍼의 값을 가져갈 수 있을 것이다.
- 그러면 락을 가진 스레드도 버퍼에서 값을 획득하거나 값을 채우고 락을 반납할 수 있을 것이다.
