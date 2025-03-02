# 순회 - Iterable, Iterator

## 1.Iterable

### 1-1. 정의
: `반복 가능한`

### 1-2. 주요 메서드
```
 public interface Iterable<T> {
 Iterator<T> iterator();
 }
```
- 단순히 `Iterator` 반복자를 반환

## 2. Iterator

### 1-1. 정의
: `반복자`

### 1-2. 주요 메서드
```
 public interface Iterator<E> {
 boolean hasNext();
 E next();
 }
```
- `hasNext()`: 다음 요소가 있는지 확인. 다음 요소가 없으면 `false`를 반환
- `next()` : 다음 요소를 반환. 내부에 있는 위치를 다음으로 이동.

## 3. 활용
- `순회` : 자료 구조에 들어있는 데이터를 차례대로 접근해서 처리하는 것
- 다양한 자료구조가 있고, 각각의 자료 구조마다 데이터를 접근하는 방법이 모두 다름
  (ex. 배열리스트는 `index`를 `size`까지 차례로 증가하면서 순회. 연결리스트는 `node.next`를 사용해서 `node`의 끝이 `null`일 때까지 순회.
- 그러나, 자료 구조에 다음 요소가 있는지 물어보고, 있으면 다음 요소를 꺼내는 과정을 반복하는 것으로 추상화 가능

## 4. 향상된 for문
- `Iterable` 인터페이스를 구현한 객체는 향상된 for 문 사용 가능
```
 for (int value : myArray) {
    System.out.println("value = " + value);
 }
```
-> 자바는 컴파일 시점에 다음과 같이 코드 변경
```
while (iterator.hasNext()) {
    Integer value = iterator.next();
    System.out.println("value = " + value);
 }
```

## 5. 자바가 제공하는 Iterable, Iterator
- 자바 `Collection` 인터페이스의 상위에 `Iterable`이 있다는 것은, 모든 컬렉션을 `Iterable`과 `Iterator`를 사용해서 순회 가능하다는 의미.
- `Map`의 경우 `Key`뿐만 아니라 `Value` 까지 있기 때문에 바로 순회는 불가능. 대신에 `Key`나 `Value`를 정해서 순회 가능.
   `keySet()`, `values()`, `entrySet()` 이용해서 순회 가능.
