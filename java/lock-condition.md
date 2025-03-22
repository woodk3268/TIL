# 1. Lock Condition

## 1-1. Condition
**Condition**
-  Condition 은 ReentrantLock을 사용하는 스레드가 대기하는 스레드 대기 공간이다.
- lock.newCondition() 메서드를 호출하면 스레드 대기 공간이 만들어진다.
- 참고로, Object.wait()에서 사용한 스레드 대기 공간은 모든 객체 인스턴스가 내부에 기본으로 가지고 있다.
- 반면에 Lock(ReentrantLock)을 사용하는 경우 이렇게 스레드 대기 공간을 직접 만들어서 사용해야 한다.

**condition.await()**
- Object.wait()와 유사한 기능이다. 지정한 condition에 현재 스레드를 대기(WAITING) 상태로 보관한다.

**condition.signal()**
- Object.notify()와 유사한 기능이다. 지정한 condition에서 대기 중인 스레드를 하나 깨운다. 깨어난 스레드는 condition에서 빠져나온다.


## 1-2. 예제 1: 스레드 대기 공간 미분리
```
public class BoundedQueueV4 implements BoundedQueue {
  private final 락 = new ReentrantLock();
  private final 스레드 대기공간 = lock.newCondition();

  private final 데이터를 보관하는 버퍼 = new ArrayDeque<>();
  private final int 버퍼에 저장할 수 있는 최대 크기;

  public BoundedQueueV4(int max) {
      this.max = max;
  }

  @Override
  public synchronized void put(String data) {
    락을 얻음;
    try{
      while(큐가 가득차면) {
           try{
                 condition.await(); // 지정한 condition에 현재 스레드를 대기(WAITING) 상태로 보관. 이때 락을 반납하고 대기 상태로 condition에 보관됨.
           }catch(InterruptedException e){}
      }
      큐에 데이터를 저장;
        condition.signal(); // 지정한 condition에서 대기 중인 스레드를 하나 깨움. 깨어난 스레드는 condition에서 빠져나옴.
      } finally {
          락을 반납;
      }
   
  }

  @Override
  public synchronized String take() {
    락을 얻음;
  try{
    while(큐가 비어있으면) {
         try{
               condition.await(); // 락을 반납하고 대기 (RUNNABLE -> WAITING)
         }catch(InterruptedException e){}
    }
    condition.signal() // 대기 중인 스레드를 깨움. WAIT -> BLOCKED
    return 큐에서 꺼낸 데이터;
  } finally {
      락을 반납;
  }

  }
}
```

## 1-3. 예제 2: 스레드 대기 공간 분리
```
public class BoundedQueueV5 implements BoundedQueue {
  private final 락 = new ReentrantLock();
  private final 소비자 스레드 대기공간 = lock.newCondition();
  private final 생산자 스레드 대기공간 = lock.newCondition();

  private final 데이터를 보관하는 버퍼 = new ArrayDeque<>();
  private final int 버퍼에 저장할 수 있는 최대 크기;

  public BoundedQueueV5(int max) {
      this.max = max;
  }

  @Override
  public synchronized void put(String data) {
    락을 얻음;
    try{
      while(큐가 가득차면) {
           try{
                 생산자 스레드 대기공간.await(); // 지정한 condition에 현재 스레드를 대기(WAITING) 상태로 보관. 이때 락을 반납하고 대기 상태로 condition에 보관됨.
           }catch(InterruptedException e){}
      }
      큐에 데이터를 저장;
        소비자 스레드 대기공간.signal(); // 지정한 condition에서 대기 중인 스레드를 하나 깨움. 깨어난 스레드는 condition에서 빠져나옴.
      } finally {
          락을 반납;
      }
   
  }

  @Override
  public synchronized String take() {
    락을 얻음;
  try{
    while(큐가 비어있으면) {
         try{
               소비자 스레드 대기공간.await(); // 락을 반납하고 대기 (RUNNABLE -> WAITING)
         }catch(InterruptedException e){}
    }
    생산자 스레드 대기공간.signal() // 대기 중인 스레드를 깨움. WAIT -> BLOCKED
    return 큐에서 꺼낸 데이터;
  } finally {
      락을 반납;
  }

  }
}
```

## 1-4. Object.notify() vs Condition.signal()
- Object.notify()
  - 대기 중인 스레드 중 임의의 하나를 선택해서 깨운다. 스레드가 깨어나는 순서는 정의되어 있지 않다.
  - synchronized 블록 내에서 모니터 락을 가지고 있는 스레드가 호출해야 한다.
- Condition.signal()
   - 대기 중인 스레드 중 하나를 깨우며, 일반적으로는 FIFO 순서로 깨운다.
   - 보통 Condition의 구현은 Queue 구조를 사용하기 때문에 FIFO 순서로 깨웁니다.
   - ReentrantLock을 가지고 있는 스레드가 호출해야 한다.
 
# 2. 스레드의 대기
## 2-1. synchronized 대기
   - 대기1 : 락 획득 대기
      - BLOCKED 상태로 락 획득 대기
      - synchronized를 시작할 때 락이 없으면 대기
      - 다른 스레드가 synchronized를 빠져나갈 때 대기가 풀리며 락 획득 시도
    - 대기2 : wait() 대기
       - WAITING 상태로 대기
       - wait()를 호출했을 때 스레드 대기 집합에서 대기
       - 다른 스레드가 notify()를 호출했을 때 빠져나감
    
## 2-2. 락 대기 집합
- 락을 기다리는 BLOCKED 상태의 스레드들을 관리한다.

## 2-3. 정리
- 자바의 모든 객체 인스턴스는 멀티스레드와 임계 영역을 다루기 위해 내부에 3가지 기본 요소를 가진다.
  - 모니터 락
  - 락 대기 집합(모니터 락 대기 집합)
  - 스레드 대기 집합
 
- 여기서 락 대기 집합이 1차 대기소이고, 스레드 대기 집합이 2차 대기소라 생각하면 된다.
- 2차 대기소에 들어간 스레드는 2차, 1차 대기소를 모두 빠져나와야 임계영역을 수행할 수 있다.

이 3가지 요소는 서로 맞물려 돌아간다.
- synchronized 를 사용한 임계 영역에 들어가려면 모니터 락이 필요하다.
- 모니터 락이 없으면 락 대기 집합에 들어가서 BLOCKED 상태로 락을 기다린다.
- 모니터 락을 반납하면 락 대기 집합에 있는 스레드 중 하나가 락을 획득하고 BLOCKED -> RUNNABLE 상태가 된다.
- wait()를 호출해서 스레드 대기 집합에 들어가기 위해서는 모니터 락이 필요하다.
- 스레드 대기 집합에 들어가면 모니터 락을 반납한다.
- 스레드가 notify()를 호출하면 스레드 대기 집합에 있는 스레드 중 하나가 스레드 대기 집합을 빠져나온다.
- 그리고 모니터 락 획득을 시도한다.
  - 모니터 락을 획득하면 임계 영역을 수행한다.
  - 모니터 락을 획득하지 못하면 락 대기 집합에 들어가서 BLOCKED 상태로 락을 기다린다.
 
## 2-4. ReentrantLock 대기
  - 대기1 : ReentrantLock 락 획득 대기
    - ReentrantLock의 대기 큐에서 관리
    - WAITING 상태로 락 획득 대기
    - lock.lock()을 호출했을 때 락이 없으면 대기
    - 다른 스레드가 lock.unlock()을 호출했을 때 대기가 풀리며 락 획득 시도, 락을 획득하면 대기 큐를 빠져나감
  - 대기2 : await() 대기
     - condition.await()을 호출했을 때, condition 객체의 스레드 대기 공간에서 관리
     - WAITING 상태로 대기
     - 다른 스레드가 condition.signal()을 호출했을 때, condition 객체의 스레드 대기 공간에서 빠져나감
   

