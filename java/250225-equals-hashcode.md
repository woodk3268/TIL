
# equals, hashCode 의 중요성

## 1. hashCode, equals를 모두 구현하지 않은 경우

1-1. hashCode() : Object의 기본기능을 사용하므로 객체의 참조값을 기반으로 해서 코드 생성

1-2. 데이터 저장문제 : 논리적으로 같은 데이터도 해시코드가 다르면 다른 위치에 각각 저장됨.

1-3. 데이터 검색문제 : 객체를 검색하기 위해, 새로운 객체를 만들면 두 객체의 해시코드가 다르므로 다른 위치에서 데이터를 찾게 됨.

## 2. hashCode는 구현했지만 equals를 구현하지 않은 경우

2-1. equals() : Object의 기본기능을 사용하므로 객체의 참조값을 기반으로 비교

2-2. 데이터 저장 문제 : hashCode()를 재정의하면, 논리적으로 같은 객체는 같은 해시코드를 사용함.
따라서 같은 해시 인덱스에 데이터가 저장됨.
그런데, 중복 데이터를 체크하는 로직에서 인스턴스의 참조값을 비교하므로 중복데이터가 없다는 결과가 나옴.
같은 인덱스에 논리적으로 같은 객체 모두 저장.

2-3. 데이터 검색 문제 : 해시 인덱스는 정확히 찾을 수 있으나, equals()에서 인스턴스의 참조값 비교하므로 비교에 실패함.

## 3. hashCode와 equals를 모두 구현한 경우

3-1. 데이터 저장 : 같은 해시 코드를 사용하면, 해시 인덱스도 같음.
중복 데이터 저장 로직에서도, 논리적으로 같은 객체는 equals()에서도 같다는 결과를 줌.

3-2. 데이터 검색 : 해시 인덱스 내부의 데이터를 모두 equlas() 비교

## 4. 정리
해시 자료 구조를 사용하는 경우 hashCode()와 equals()를 반드시 함께 재정의해야 함.
