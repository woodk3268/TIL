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

## 2. 예제

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
