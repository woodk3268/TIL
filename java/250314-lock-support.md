# 1. LockSupport
## 1-1. 정의
- LockSupport는 스레드를 WAITING 상태로 변경한다.
- WAITING 상태는 누가 깨워주기 전까지는 계속 대기한다. 그리고 CPU 실행 스케줄링에 들어가지 않는다.

## 1-2. 기능
- park() : 스레드를 WAITING 상태로 변경한다.
- parkNanos(nanos) : 스레드를 나노초 동안만 TIMED_WAITING 상태로 변경한다.(지정한 나노초가 지나면 RUNNABLE 상태로 변경된다.)
- unpark(thread) : WAITING 상태의 대상 스레드를 RUNNABLE 상태로 변경한다.

## 1-3. 예제
### 1-3-1. unpark 사용
- main 스레드가 Thread-1을 start()하면 Thread-1은 RUNNABLE 상태가 된다.
- Thread-1은 Thread.park()를 호출한다. Thread-1은 RUNNABLE -> WAITING 상태가 되면서 대기한다.
- main 스레드가 Thread-1을 unpark()로 깨운다. Thread-1은 WAITING -> RUNNABLE 상태로 변한다.
- 이처럼, LockSupport는 특정 스레드를 WAITING 상태로, 또 RUNNABLE 상태로 변경할 수 있다.

### 1-3-2. 인터럽트 사용
- WAITING 상태의 스레드에 인터럽트가 발생하면 WAITING 상태에서 RUNNABLE 상태로 변하면서 깨어난다.
- BLOCKED 상태와 달리 WAITING 상태는 인터럽트가 걸리면 대기 상태를 빠져나온다.

## 1-4. BLOCKED vs WAITING
### 1-4-1. BLOCKED 용도
- 자바의 synchronized에서 락을 획득하기 위해 대기할 때 사용된다.

### 1-4-2. WAITING 용도
- WAITING, TIMED_WAITING 상태는 스레드가 특정 조건이나 시간 동안 대기할 때 발생하는 상태이다.
- 예를 들어, Thread.join(), LockSupport.part(), Object.wait()와 같은 메서드 호출 시 WAITING 상태가 된다.
- TIMED_WAITING 상태는 Thread.sleep(ms), Object.wait(long timeout), Thread.join(long millis), LockSupport.parkNanos(ns) 등과 같은 시간 제한이 있는 대기 메서드를 호출할 때 발생한다.

### 1-4-3. 정리
- 모두 스레드가 대기하며, 실행 스케줄링에 들어가지 않기 때문에, CPU 입장에서 보면 실행하지 않는 비슷한 상태이다.
- BLOCKED 상태는 synchronized 에서만 사용하는 특별한 대기 상태이다.
- WAITING, TIMED_WAITING 상태는 범용적으로 활용할 수 있는 대기 상태이다.

## 1-5. LockSupport 정리
- LockSupport를 사용하면 스레드를 WAITING, TIMED_WAITING 상태로 변경할 수 있고, 또 인터럽트를 받아서 스레드를 깨울 수도 있다.
- 이처럼, LockSupport를 활용하면, 무한 대기하지 않는 락 기능을 만들 수도 있다.
  - 락(lock)이라는 클래스를 만들고, 특정 스레드가 먼저 락을 얻으면 RUNNABLE로 실행하고, 락을 얻지 못하면 park()를 사용해서 대기 상태로 만든느 것이다.
  - 그리고 스레드가 임계 영역의 실행을 마치고 나면 락을 반납하고, unpark()를 사용해서 대기 중인 다른 스레드를 깨우는 것이다.
  - 물론 parkNanos()를 사용해서 너무 오래 대기하면 스레드가 스스로 중간에 깨어나게 할 수도 있다.
