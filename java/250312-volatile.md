# volatile

## 1. 예제
- main 스레드와 work 스레드 간 공유하는 변수에 volatile 키워드 미사용
- ``` boolean runFlag = true; ```

- main 스레드는 새로운 스레드인 work 스레드를 생성하고 작업을 시킨다.
- work 스레드는 run() 메서드를 실행하면서 while(runFlag)가 true인 동안 계속 작업을 한다.
- 만약 runFlag가 false로 변경되면 반복문을 빠져나오면서 "task 종료"를 출력하고 작업을 종료한다.
- main 스레드는 sleep()을 통해 1초간 쉰 다음에 runFlag를 false로 설정한다.
- work 스레드는 run() 메서드를 실행하면서 while(runFlag)를 체크하는데, 이제 runFlag가 false가 되었으므로 "task 종료"를 출력하고 작업을 종료해야 한다.
- 
- 그러나 실제 실행 결과를 보면 task 종료가 출력되지 않는다.
- work 스레드가 while문에서 빠져나오지 못하고 있다.

## 2. volatile, 메모리 가시성
- 일반적으로 생각하는 메모리 접근 방식
- main 스레드와 work 스레드는 각각의 CPU 코어에 할당되어서 실행된다.
- 자바 프로그램을 실행하고 main 스레드와 work 스레드는 모두 메인 메모리의 runFlag의 값을 읽는다.
- 프로그램의 시작 시점에는 runFlag를 변경하지 않기 때문에 모든 스레드에서 true의 값을 읽는다.
- work 스레드의 경우 while(runFlag[true])가 만족하기 때문에 while문을 계속 반복해서 수행한다.
  - main 스레드는 runFlag 값을 false로 설정한다.
  - 이때 메인 메모리의 runFlag 값이 false로 설정된다.
  - work 스레드는 while(runFlag)를 실행할 때 runFlag의 데이터를 메인 메모리에서 확인한다.
  - runFlag의 값이 false이므로 while문을 탈출하고, "task 종료"를 출력한다.
