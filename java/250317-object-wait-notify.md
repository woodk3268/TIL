# 1. Object - wait, notify
- 자바는 처음부터 멀티스레드를 고려하며 탄생한 언어다.
- synchronized를 사용한 임계 영역 안에서 락을 가지고 무한 대기하는 문제는 Object 클래스의  wait(), notify() 로 해결 가능하다.

## 1-1. wait(), notify() 설명
- Object.wait()
   - 현재 스레드가 가진 락을 반납하고 대기(WAITING)한다.
   - 현재 스레드를 대기(WAITING) 상태로 전환한다. 
   - 이 메서드는 현재 스레드가 synchronized 블록이나 메서드에서 락을 소유하고 있을 때만 호출할 수 있다.
   - 호출한 스레드는 락을 반납하고, 다른 스레드가 해당 락을 획득할 수 있도록 한다.
   - 이렇게 대기 상태로 전환된 스레드는 다른 스레드가 notify() 또는 notifyAll()을 호출할 때까지 대기 상태를 유지한다.
- Object.notify()
  - 대기 중인 스레드 중 하나를 깨운다.
  - 이 메서드는 synchronized 블록이나 메서드에서 호출되어야 한다.
  - 깨운 스레드는 락을 다시 획득할 기회를 얻게 된다.
  - 만약 대기 중인 스레드가 여러 개라면, 그 중 하나만이 깨워지게 된다.
- Object.notifyAll()
  - 대기 중인 모든 스레드를 깨운다.
  - 이 메서드 역시 synchronized 블록이나 메서드에서 호출되어야 한다.
  - 모든 대기 중인 스레드가 락을 획득할 수 있는 기회를 얻게 된다.

## 1-2. 예제
### 1-2-1. 버퍼 구현체
```
public class BoundedQueueV3 implements BoundedQueue {
  private final 데이터를 보관하는 버퍼 = new ArrayDeque<>();
  private final int 버퍼에 저장할 수 있는 최대 크기;

  public BoundedQueueV1(int max) {
      this.max = max;
  }

  @Override
  public synchronized void put(String data) {
    while(큐가 가득차면) {
         try{
               wait(); // 락을 반납하고 대기 (RUNNABLE -> WAITING)
         }catch(InterruptedException e){}
    }
    큐에 데이터를 저장;
      notify(); // 대기 중인 스레드를 깨움. WAIT -> BLOCKED
  }

  @Override
  public synchronized String take() {
    while(큐가 비어있으면) {
         try{
               wait(); // 락을 반납하고 대기 (RUNNABLE -> WAITING)
         }catch(InterruptedException e){}
    }
    notify() // 대기 중인 스레드를 깨움. WAIT -> BLOCKED
    return 큐에서 꺼낸 데이터;
  }
}
```

** 스레드 대기 집합(wait set)**
- synchronized 임계 영역 안에서 Object.wait()를 호출하면 스레드는 대기(WAITING) 상태에 들어간다.
- 이렇게 대기 상태에 들어간 스레드를 관리하는 것을 대기 집합이라 한다.
- 참고로 모든 객체는 각자의 대기 집합을 가지고 있다.
- 모든 객체는 락(모니터 락)과 대기 집합을 가지고 있다. 둘은 한 쌍으로 사용된다.
- 따라서, 락을 획득한 객체의 대기 집합을 사용해야 한다.

## 1-3. 한계
- 지금까지 살펴본 Object.wait(), Object.notify() 방식은 스레드 대기 집합 하나에 생산자, 소비자 스레드를 모두 관리한다.
- 그리고 notify()를 호출할 때 임의의 스레드가 선택된다.
- 따라서, 큐에 데이터가 없는 상황에 소비자가 같은 소비자를 깨우는 비효율이 발생할 수도 있다.
- 또는 큐에 데이터가 가득 차있는데 생산자가 같은 생산자를 깨우는 비효율도 발생할 수 있다.
- 같은 종류의 스레드를 깨울 때 비효율이 발생한다.
- 또한, 어떤 스레드가 깨어날 지 알 수 없기 때문에 스레드 기아 문제가 발생한다.
- 대기 상태의 스레드가 실행 순서를 계속 얻지 못해서 실행되지 않는 상황을 스레드 기아 상태라고 한다.
- 다만, notifyAll()을 사용하면 스레드 대기 집합에 있는 모든 스레드를 한번에 다 깨울 수 있다.
- notifyAll()을 사용해서 스레드 기아 문제는 막을 수 있지만, 비효율을 막지는 못한다.

