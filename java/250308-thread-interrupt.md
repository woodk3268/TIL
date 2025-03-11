# 1. 특정 스레드의 작업을 중단하는 방법

## 1-1. 변수 사용
- runFlag를 사용해서 work 스레드에 작업 중단 지시
- 프로그램 시작 후 4초 뒤에 main 스레드는 runFlag를 false로 변경
- work 스레드는 while(runFlg)에서 runFlag의 조건이 false로 변한 것을 확인하고, while 문을 빠져나가면서 작업 종료
- 문제점 : main 스레드가 runFlag = false를 통해 작업 중단을 지시해도, work 스레드가 sleep()을 통해 잠들어 있는 상태이면 즉각 반응하지 않음.

## 1-2. interrupt 사용
- 특정 스레드가 Thread.sleep()을 통해 쉬고 있는데, 처리해야 하는 작업이 들어와서 해당 스레드를 급하게 깨워야 하는 경우
- 또는 sleep()으로 쉬고 있는 스레드에게 더는 일이 없으니, 작업 종료 지시하는 경우
- WAITING, TIMED_WAITING 같은 대기 상태의 스레드를 직접 깨워서, 작동하는 RUNNABLE 상태로 만듦

### 1-2-1. 동작 방식
- 특정 스레드의 interrupt() 메서드를 호출하면, 해당 스레드에 인터럽트 발생
- 인터럽트가 발생하면, 해당 스레드에 InterruptedException 발생
   - 이때 인터럽트를 받은 스레드는 대기 상태에서 깨어나 RUNNABLE 상태가 되고, 코드를 정상 수행
   - InterruptedException을 catch로 잡아서 정상흐름으로 변경
- 참고로, Thread.sleep()처럼 InterruptedException 을 던지는 메서드를 호출하거나 또는 호출하여 대기중일 때 예외 발생

### 1-2-2. 예제
- main 스레드가 4초 뒤에 work 스레드에 interrupt()를 검.
- work 스레드는 인터럽트 상태(true)가 됨.
- 스레드가 인터럽트 상태일 때는, sleep() 처럼 InterruptedException이 발생하는 메서드를 호출하거나 또는 이미 호출하고 대기중이라면 InterruptedException이 발생
  - work 스레드는 TIMED_WAITING 상태에서 RUNNABLE 상태로 변경되고, InterruptedException 예외를 처리하면서 반복문 탈출
  - work 스레드는 인터럽트 상태가 되었고, 인터럽트 상태이기 때문에 인터럽트 예외 발생
  - 인터럽트 상황에서 인터럽트 예외가 발생하면 work 스레드는 다시 작동하는 상태가 됨. 따라서 work 스레드의 인터럽트 상태는 종료됨.(인터럽트 상태 : false로 변경)
