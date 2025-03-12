# 인터럽트 예제 
- 사용자의 입력을 프린터에 출력하는 예제
- 사용자의 입력을 받는 main 스레드와 사용자의 입력을 출력하는 printer 스레드
 - main 스레드 : 사용자의 입력을 받아서 Printer 인스턴스의 jobQueue에 담는다.
 - printer 스레드 : jobQueue가 있는지 확인한다.
  -jobQueue에 내용이 있으면 poll()을 이용해서 꺼낸 다음에 출력한다.
   - 출력을 완료하면 while문을 다시 반복한다.
   - 만약 jobQueue가 비었다면 continue 를 사용해서 다시 while문을 반복한다.
   - 이렇게 해서 jobQueue에 출력할 내용이 들어올 때까지 계속 확인한다.

# 1. interrupt() 미사용
- main 스레드 : 사용자가 q를 입력하면, printer.work의 값을 false로 변경한다.
  - main 스레드는 while문을 빠져나가고 main 스레드가 종료된다.
- printer 스레드 : while 문에서 work의 값이 false인 것을 확인한다.
  - printer 스레드는 while문을 빠져나가고, "프린터 종료"를 출력하고, printer 스레드는 종료된다.

문제점 : printer 스레드가 sleep(3000)을 통해 대기 상태에서 빠져나와야 while문 체크하므로
종료(q)를 입력했을 때 바로 반응하지 않음.

# 2. interrupt() 도입 - work 변수 사용
- 종료(q) 입력 시, main 스레드가 work 변수도 false로 변경하고 , printer 스레드에 인터럽트도 함께 호출한다.
- interrupt() : sleep() 상태에서 빠져나온다.
- work = false : while 문을 체크하는 곳에서 빠져나온다.

# 3. interrupt() 도입 - work 변수 제거
- Thread.interrupted() 메서드를 사용해 해당 스레드가 인터럽트 상태인지 아닌지 확인
