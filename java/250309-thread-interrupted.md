# 인터럽트 코드 개선

## 1. while(true){}
### 1-1. 개선할 점
- 인터럽트가 발생해도 while(true) 부분은 true이기 때문에 다음 코드로 넘어감. => while(인터럽트_상태_확인)으로 변경
- sleep()을 호출하고 나서야 인터럽트 발생 => sleep()과 같은 코드가 없어도 인터럽트 상태 직접 확인해서 while문 빠져나가도록 변경

## 2. while(인터럽트_상태_확인)
- Thread.currentThread().isInterrupted()
  - Thread.currentThread() : 이 코드를 실행하는 스레드 조회 가능
  - isInterrupted() : 스레드가 인터럽트 상태인지 확인 가능

### 2-1. 예제
- main 스레드는 interrupt() 메서드를 사용해서, work 스레드에 인터럽트를 검
- work 스레드는 인터럽트 상태(isInterrupted()=true)
- while 조건이 false가 되면서 while 문 탈출
- work 스레드의 인터럽트 상태가 true로 계속 유지됨. 
  cf. 앞서 인터럽트 예외가 터진 경우 스레드의 인터럽트 상태는 false 가 됨
- 이때 만약 인터럽트가 발생하는 sleep()과 같은 코드를 수행한다면, 해당 코드에서 인터럽트 예외 발생
- 자원 정리를 하는 도중에 인터럽트가 발생해서, 자원 정리에 실패
- 자바에서 인터럽트 예외가 한 번 발생하면, 스레드의 인터럽트 상태를 다시 정상으로 돌리는 것은 이런 이유

## 3.while(인터럽트_상태_확인) 및 인터럽트 상태 변경
- main 스레드는 interrupt() 메서드를 사용해서, work 스레드에 인터럽트를 검
- work 스레드는 인터럽트 상태(interrupted()=true)
   - 이때, Thread.interrupted()는 스레드가 인터럽트 상태(true)라면, true를 반환하고, 해당 스레드의 인터럽트 상태를 false로 변경
- while문을 탈출하는 시점에, 스레드의 인터럽트 상태도 false로 변경됨
- work 스레드는 이후에 자원을 정리하는 코드를 실행하는데, 이때 인터럽트의 상태를 false이므로 인터럽트가 발생하는 sleep()과 같은 코드를 수행해도 인터럽트 발생하지 않음.
