# 동시성 컬렉션
## 1. 동시성 컬렉션이 필요한 이유
- 여러 스레드가 하나의 인스턴스에 동시에 접근해도 괜찮은 경우를 스레드 세이프(Thread safe)하다고 한다.
- 그렇다면 ArrayList는 스레드 세이프 할까?

### 1-1. 컬렉션 직접 만들기

```
public interface SimpleList {

    void add(Object e);
}
```

```
public class BasicList implements SimpleList {
      private static final int DEFAULT_CAPACITY = 5;

      private Object[] 데이터를 담을 배열;
      private int 배열의 크기;

      public BasicList() {
        배열 = new Object[DEFAULT_CAPACITY];
      }

      @Override
      public void add(Object 추가할 원소) {
        배열에 데이터 추가;
        배열의 크기 증가;    
      }

}
```

### 1-2. 멀티스레드 문제 확인
**add() - 원자적이지 않은 연산**
```
public void add(Object e){
    배열에 데이터 추가;
    배열의 크기 증가;
}
```

### 1-3. 동시성 문제
```
public class SimpleListMain {
  public static void main(String[] args) throws InterruptedException {
       test(new BasicList());
    }
  private static void test(SimpleList list) throws InterruptedException {
      Runnable addA = new Runnable() {
                @Override
                public void run() {
                  list.add("A");
            }
        };
      Runnable addB = new Runnable() {
              @Override
               public void run() {
                   list.add("B");
            }
        };
        Thread thread1 = new Thread(addA, "Thread-1");
        Thread thread2 = new Thread(addB, "Thread-2");
        thread1.start();
        thread2.start();
    }
 }
```
- 스레드 1, 스레드2가 배열에 데이터 추가하는 코드를 동시에 수행한다.
- 만약 스레드 1이 약간 빠르게 수행하면,
    - 스레드 1 수행 : elementData[0] = A, elementData[0]의 값은 A가 된다.
    - 스레드 2 수행 : elementData[0] = B, elementData[0]의 값은 B가 된다.
- 이후 배열의 크기를 증가시키는 과정에서, 2가지 상황이 발생할 수 있다.
    - 스레드 1이 스레드2보다 약간 빠르게 수행한다.
      - 스레드1 수행 : size++, size의 값은 1이 된다.
      - 스레드2 수행 : size++, size의 값은 1->2가 된다.
      - > 결과적으로 size의 값은 2가 된다.
    - 스레드1, 스레드2 가 거의 동시에 실행된다.
       - 스레드1(스레드2) 수행 : size = size + 1 연산이다. size의 값을 읽는다. 0이다.
       - 스레드1(스레드2) 수행 : size = 0 + 1 연산을 수행한다.
       - 스레드1(스레드2) 수행 : size = 1 대입을 수행한다.
       - > 결과적으로 size의 값은 1이 된다.

- 자료구조들은 단순한 연산을 제공하는 것처럼 보인다.
- 예를 들어, 데이터를 추가하는 add()와 같은 연산은 마치 원자적인 연산처럼 느껴진다.
- 하지만, 배열에 데이터를 추가하고, 사이즈를 변경하고, 배열을 새로 만들어서 배열의 크기도 늘리고, 노드를 만들어서 링크에 연결하는 등 수많은 복잡한 연산이 함께 사용된다.
- 따라서, 일반적인 컬렉션들은 절대로 스레드 세이프하지 않다.

### 1-4. 동기화
- 앞서 만든 BasicList에 synchronized 키워드를 추가한다.
- add() 메서드에 synchronized 를 통해 안전한 임계 영역을 만들었기 때문에, 한 번에 하나의 스레드만 add() 메서드를 수행한다.
- 실행 순서
   - 스레드 1,2가 add() 코드를 동시에 수행한다. 스레드1이 약간 빠르게 수행하는 것을 가정한다.
      - 스레드1 수행
        - 락을 획득한다.
        - 배열에 데이터를 추가하고 크기를 증가시킨다.
      - 스레드2 수행
         - 스레드1이 가져간 락을 획득하기 위해 BLOCKED 상태로 대기한다.
         - 스레드1이 락을 반납하면 락을 획득한다.
         - 배열에 데이터를 추가하고 크기를 증가시킨다.
       
### 1-5. 프록시 도입
- ArrayList, LinkedList,HashSet, HashMap 등의 코드도 모두 복사해서 synchronized 기능을 추가한 코드를 만들어야 할까?
- 하지만 이렇게 코드를 복사해서 만들면 이후에 구현이 변경될 때 같은 모양의 코드를 2곳에서 변경해야 한다.
- 기존 코드를 그대로 사용하면서 멀티스레드 상황에 동기화가 필요할 때만 synchronized 기능을 살짝 추가하고 싶다면 어떻게 하면 될까?
- 이럴때 사용하는 것이 프록시이다.

```
public class SyncProxyList implements SimpleList {
    private SimpleList 실제 호출되는 대상;

    public SyncProxyList(SimpleList target){
      실제 호출되는 대상을 주입받음;

    }

     @Override
    public synchronized void add(Object e) {
        실제 호출되는 대상의 add(e) 호출;
    }
}
```
- 이 클래스의 역할은 모든 메서드에 synchronized를 걸어주는 일 뿐이다. 그리고나서 실제 호출되는 대상의 같은 기능을 호출한다.

### 1-6. 프록시 정리
- 프록시인 SyncProxyList는 원본인 BasicList와 똑같은 SimpleList를 구현한다.
- 따라서 클라이언트인 test() 입장에서는 원본 구현체가 전달되든, 아니면 프록시 구현체가 전달되든 아무런 상관이 없다.
- 단지 수많은 SimpleList의 구현체 중의 하나가 전달되었다고 생각할 뿐이다.
- 프록시는 내부에 원본을 가지고 있다. 그래서 프록시가 필요한 일부의 일을 처리하고, 그다음에 원본을 호출하는 구조를 만들 수 있다.
- 여기서 프록시는 synchronized를 통한 동기화를 적용한다.

## 2. 자바 동시성 컬렉션
- 자바가 제공하는 java.util 패키지에 있는 컬렉션 프레임워크들은 대부분 스레드 세이프하지 않다.
- 처음부터 모든 자료 구조에 synchronized를 사용해서 동기화를 해두면 어떨까?
- synchronized, Lock, CAS 등 모든 방식은 성능과 트레이드 오프가 있다.
- 결국 동기화를 사용하지 않는 것이 가장 빠르다.
- 좋은 대안으로는 synchronized를 적용하는 프록시를 만드는 방법이 있다.
- 자바는 컬렉션을 위한 프록시 기능을 제공한다.

## 2-1. synchronized 프록시 방식의 단점
- 첫째, 동기화 오버헤드가 발생한다. 각 메서드 호출 시마다 동기화 비용이 추가된다.
- 둘째, 전체 컬렉션에 대해 동기화가 이루어지기 때문에, 잠금 범위가 넓어질 수 있다. 병렬 처리의 효율성을 저하시키는 요인이 된다.
- 셋째, 정교환 동기화가 불가능하다. 특정 부분이나 메서드에 대해 선택적으로 동기화를 적용하는 것은 어렵다.
