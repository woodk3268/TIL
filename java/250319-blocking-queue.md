# 1. BlockingQueue
: 자바는 생산자 소비자 문제를 해결하기 위해 java.util.concurrent.BlockingQueue라는 특별한 멀티스레드 자료구조를 제공한다.
이것은 이름 그대로 스레드를 차단(Blocking) 할 수 있는 큐다.

- 데이터 추가 차단 : 큐가 가득 차면 데이터 추가 작업(put())을 시도하는 스레드는 공간이 생길 때까지 차단된다.
- 데이터 획득 차단 : 큐가 비어 있으면 획득 작업(take())을 시도하는 스레드는 큐에 데이터가 들어올 때까지 차단된다.

## 1-1. 기능
  - 실무에서 멀티스레드를 사용할 때는 응답성이 중요하다.
  - 예를 들어서 대기 상태에 있어도, 고객이 중지 요청을 하거나, 또는 너무 오래 대기한 경우 포기하고 빠져나갈 수 있는 방법이 필요하다.
  - 큐가 가득 찼을 때 생각할 수 있는 선택지는 4가지이다.
    - 예외를 던진다. 예외를 받아서 처리한다.
    - 대기하지 않는다. 즉시 false를 반환한다.
    - 대기한다.
    - 특정 시간 만큼만 대기한다.
   
## 1-2. 즉시 반환
```
public class BoundedQueueV6_2 implements BoundedQueue {

 private BlockingQueue<String> queue;

  public BoundedQueueV6_2(int max) {
    queue = new ArrayBlockingQueue<>(max);
  }

  public void put(String data) {
     boolean result = queue.offer(data);
  }

  public String take() {
    return queue.poll();
  }
}
```
- offer(data) : 성공하면 true를 반환하고, 버퍼가 가득 차면 즉시 false를 반환한다.
- poll() : 버퍼에 데이터가 없으면 즉시 null 을 반환한다.
- 두 메서드는 스레드가 대기하지 않는다.

## 1-3. 시간 대기
```
public class BoundedQueueV6_3 implements BoundedQueue {

 private BlockingQueue<String> queue;

  public BoundedQueueV6_3(int max) {
    queue = new ArrayBlockingQueue<>(max);
  }

  public void put(String data) {
    try{
          boolean result = queue.offer(data, 1, TimeUnit.NANOSECONDS);
      } catch (InterruptedException e) {}
  }

  public String take() {
    try {
        return queue.poll(2, TimeUnit.SECONDS);
      } catch (InterruptedException e) {}
  }
}
```
- offer(data, 시간)는 성공하면 true를 반환하고, 버퍼가 가득 차서 스레드가 대기해야 하는 상황이면, 지정한 시간까지 대기한다.
- 대기 시간을 지나면 false를 반환한다.
- poll(시간) 버퍼에 데이터가 없어서 스레드가 대기해야 하는 상황이면 ,지정한 시간까지 대기한다.
- 대기 시간을 지나면 null 을 반환한다.

## 1-4. 예외
```
public class BoundedQueueV6_4 implements BoundedQueue {

 private BlockingQueue<String> queue;

  public BoundedQueueV6_4(int max) {
    queue = new ArrayBlockingQueue<>(max); 
  }

  public void put(String data) {
      queue.add(data); // java.lang.IllegalStateException: Queue full
  }

  public String take() {
    return queue.remove(); // java.util.NoSuchElementException
  }
}
```
- add(data)는 성공하면 true를 반환하고, 버퍼가 가득 차면 즉시 예외가 발생한다.
- remove()는 버퍼에 데이터가 없으면, 즉시 예외가 발생한다.

## 1-5. 대기
```
public class BoundedQueueV6_1 implements BoundedQueue {

 private BlockingQueue<String> queue;

 public BoundedQueueV6_2(int max) {
    queue = new ArrayBlockingQueue<>(max);
  }

  public void put(String data) {
    try{
        queue.put(data);
      } catch (InterruptedException e) {}
  }

  public void String take() {
    try{
        return queue.take();
      } catch (InterruptedException e) {}
  }
}
```
