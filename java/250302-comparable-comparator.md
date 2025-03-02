# 정렬 - Comparable, Comparator

## 1. 배열 정렬

### 1-1. Arrays.sort() 메서드 사용

### 1-2. 비교자 -`Comparator`
- 비교 기준 직접 제공 가능
- `Arrays.sort()`를 사용할 때, 비교자를 넘겨주면 알고리즘에서 비교자 사용

## 2. 직접 만든 객체 정렬

### 2-1. `Comparable` 인터페이스
- 객체에 비교 기능 추가
```
 public interface Comparable<T> {
 public int compareTo(T o);
 }
```
- 자기 자신과 인수로 넘어온 객체를 비교해서 반환
- 현재 객체가 인수로 주어진 객체보다 더 작으면 음수(ex. -1), 같으면 0, 더 크면 양수(ex. 1)

### 2-2. `Arrays.sort()` 메서드 사용
- 기본 정렬 시도

### 2-3. 비교자 -`Comparator`
- 객체가 가지고 있는 `Comparable` 기본 정렬 외 다른 정렬 방법 사용
- `Arrays.sort()`를 사용할 때, 비교자를 넘겨주면 알고리즘에서 비교자 사용

## 3. List 자료 구조 정렬
- `list.sort(null) : `Comparable`로 비교해서 정렬. 자연적인 순서로 비교
- `list.sort(new IdComparator()) : 전달한 비교자로 비교 

## 4. Tree 구조 정렬
- `TreeSet`과 같은 이진 탐색 트리 구조는 데이터를 정렬하면서 보관-> 정렬 기준을 제공하는 것이 필수
- 왼쪽 노드에 저장해야 할지, 오른쪽 노드에 저장해야 할지 비교 필요.
- 따라서 `TreeSet`, `TreeMap` 은 `Comparable` 또는 `Comparator`가 필수

### 4-1. 생성 방법
- `new TreeSet<>()`: `TreeSet`을 생성할 때, 별도의 비교자를 제공하지 않으면 객체가 구현한 `Comparable` 사용
- `new TreeSet<>(new IdComparator()) : 별도의 비교자를 제공하면 `Comparable` 대신 비교자(`Comparator`)를 사용해서 정렬
