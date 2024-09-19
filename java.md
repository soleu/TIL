### 1. 기본형 변수, 참조형 변수의 차이점

기본형 변수 : Primate Type

- 총 8가지 (정수형 : int, long, short, char, byte / 실수형 : float, double / boolean)

- 값 복사 (call by value) - 값을 복사. 다른 메서드에서 변경을 해도 원래 변수는 변경이 안됨

참조형 변수 : Reference Type

- 주소 복사 (call by reference) - 주소를 복사. 다른 메서드에서 변경을 하면 원래 변수가 변경이 됨
- Null 허용. 초기화시 기본 값이 없으면 Null 로 저장이됨



### 2. 깊은 복사 (deep copy) / 얕은 복사(shallow copy)

깊은 복사는 값 자체를 복사하지만, 얕은 복사는 주소를 복사하게 되어 값에 변경이 일어날 수 있다.

- 얕은 복사 사용 예제

    - ```java
    Person person = new Person("John", 30);
    Person copyPerson = person;
    ```



- 깊은 복사 사용 예제

    - .clone() - 하위 객체까지는 복사를 못함

    - 생성자 사용

    - ```java
    Person person = new Person("John", 30);
    // 생성자 사용
    Person copyPerson = new Person(person.name, person.age);
    
    // clone
    Person clonePerson = person.clone();
    ```



### 3. 컬렉션 프레임워크란

자바에서 **데이터 그룹을 저장, 관리 및 처리하기 위한 클래스**들의 집합.

주요 인터페이스  - List, Map, Set

- List : 중복 허용. 순서가 있음
    - LinkedList - 양방향 포인터 구조. 스택, 큐, 양방향 큐 만들때 사용
    - Vector - 이전에는 대용량 처리를 위해 사용. 내부에서 자동 동기화 처리. 성능 안좋음
    - ArrayList - 단방향 포인터 구조. 조회 기능에 뛰어남
- Set : 중복 불가. 순서 없음
    - HashSet : 가장 빠름. 순서 예측 불가
    - TreeSet : 정렬 방법 지정 가능
- Map : key-value의 쌍을 이룸. key는 중복이 될 수 없음
    - Hashtable : 동기화 지원. null 불가
    - HashMap : 중복 불가, 순서 예측 불가. null 가능
    - TreeMap : 정렬된 순서대로 키-값을 저장. 검색 빠름

<img width="593" alt="image" src="https://github.com/user-attachments/assets/7bccdc9a-9ca7-4340-9715-e1847c41a8e8">

### 접근 제어자

클래스에서 접근 제어자를 표시하면 해당 클래스의 접근 범위가 제어됨. 접근 제어자로는 public, protected, package-private(default), private가 있고 각각의 접근 범위에 따라 외부에서 클래스에 접근할 수 있는지 여부가 결정됨. 이를 통해 캡슐화를 구현하고 외부로 부터 보호할 수 있음

- private : 해당 클래스 내에서만 사용 가능
- protected : 같은 패키지 내에서나 상속받은 클래스 내에서만 사용 가능
- package-private : 같은 패키지 내에서 사용 가능
- public : 어디서나 사용 가능

#### 래퍼 클래스

기본타입의 데이터를 객체로 표현해야 하는 경우가 종종 생김

래퍼 클래스 -> 기본형을 객체로 다루기 위해서 사용하는 클래스

Wrapper 클래스를 사용하는 이유는 뭘까?

- 래퍼 클래스는 **기본 데이터 타입을 Object로 변환할 수 있다**. 메소드에 전달된 인수를 수정하려는 경우 오브젝트가 필요하다. ( 기본 유형은 값에 의한 변경 Object는 참조에 의한 변경이기 때문이다. )
- java.util 패키지의 **클래스는 객체만 처리**하므로 Wrapper class는 이 경우에도 도움이 된다.
- ArrayList 등과 같은 **Collection** **프레임** **워크의** **데이터** **구조**는 기본 타입이 아닌 객체만 저장하게 되고 Wrapper Class를 사용하여 자동 방식과 언방식이 일어 난다.


### 자바 메모리 구조

<img width="594" alt="image" src="https://github.com/user-attachments/assets/2abdfa34-967d-46e9-bda6-e57facdd74bf">

- **Method Area**
    - 클래스 정보, 메서드의 바이트코드, 상수, static 변수 등 프로그램이 실행되면서 필요로 하는 메타데이터를 저장하는 곳
    - JVM이 실행되면서 생기는 공간
    - Class 정보, 전역변수 정보, Static 변수 정보가 저장되는 공간
    - Runtime Constant Pool - '상수'정보가 저장되는 공간
    - 모든 스레드에서 정보가 공유됨
    - 클래스 언로드 : 특정 클래스가 더 이상 사용되지 않거나, 해당 클래스의 인스턴스가 존재하지 않으며 GC의 대상이 되었을때 발생 (주로 종료될때까지 메모리에 유지)
- **Heap**
    - new 예약어로 생성된 객체 (Reference Type) 및 배열이 저장되는 공간
    - GC가 처리하지 않는 한 소멸되지 않음
        - GC에 의해 자동으로 이루어짐
    - 모든 스레드에서 정보가 공유됨
- **Stack**
    - 지역변수, 메소드의 매개변수와 같이 잠시 사용되고 없어지는 데이터가 저장되는 공간
    - LIFO
    - 지역변수이지만 참조형이라면 Heap에 저장된 데이터의 주소값을 Stack에 저장
    - 스레드마다 하나씩 존재

<img width="568" alt="image" src="https://github.com/user-attachments/assets/1aa53567-fbfd-4eca-8805-c276ba0636e8">

- PC Register
    - 스레드가 생성되면서 생기는 공간
    - 스레드가 어느 명령어를 처리하고 있는지 그 주소를 등록
    - JVM이 실행하고 있는 현재 위치를 저장하는 역할
- Native Method Stack
    - Java가 아닌 다른 언어로 구성된 메소드를 실행이 필요할 때 사용되는 공간

** 스택 메모리가 가득차면 java.lang.StackOverFlowError 발생

힙 메모리가 가득차면 java.lang.OutOfMemoryError 발생

** 스택 메모리 사이즈 << 힙 메모리

스택 메모리는 간단한 메모리 할당 방법(LIFO)를 사용하므로 힙 메모리보다 빠름

#### GC (Garbage Collector)

- 마크 앤 스윕 : 메모리 관리에서 가장 기본적인 방식

    - Mark : GC가 루트에서부터 시작해 참조되는 모든 객체를 탐색하고, 도달 가능한 객체를 마킹
    - Sweep : 마크되지 않은, 즉 도달 불가능한 객체를 메모리에서 해제
    - 장점
        - 단순함
        - 효율적인 메모리 해제
    - 단점
        - 메모리 단편화 : 스윕 후 남은 메모리 공간이 연속적이지 않을 수 있음
        - 스톱 더 월드 : 모든 작업을 중지하고 GC가 실행되므로, 시스템의 성능에 일시적인 영향을 줌

- 마크 앤 컴팩트 : 마크 앤 스윕과 비슷하지만, 메모리 단편화를 해결하기 위해 살아있는 객체들을 한 곳으로 밀어 넣음

    - Mark : GC가 루트에서부터 시작해 참조되는 모든 객체를 탐색하고, 도달 가능한 객체를 마킹
    - Compact : 살아 있는 객체들을 한쪽으로 밀어 연속적인 메모리 공간 확보
    - 장점
        - 메모리 단편화 해결
    - 단점
        - 비용이 높은 Compaction : 성능 오버헤드가 발생할 수 있음
        - 스톱 더 월드

- 제너럴 GC : 객체를 생애 주기를 기반으로 메모리 관리

    - Young Generation : 대부분의 객체가 생성된 후 금방 사용되지 않기 때문에 주로 빠른 복사 알고리즘 사용
    - Old Generation : 오래 살아남은 객체는 주고 Mark-and-Sweep 또는 Mark-and-Compact 알고리즘 사용
    - 장점
        - 최적화된 수집
        - Young Generation에서의 빠른 수집
    - 단점
        - Old Generation 수집의 비효율성
        - Stop-the-world 문제 (Old generation 에서 발생)

- 복사 알고리즘(Copying GC) : 힙 영역을 두개의 동일한 크기 영역(From Space, To Space)으로 나눈 후, 살아 있는 객체를 하나의 영역에서 다른 영역으로 복사하는 방식

    - 복사 : 루트에서 참조되는 객체들을 탐색하여 다른 영역으로 복사한 후, 기존 영역을 모두 해제
    - 장점
        - 빠른 할당 : 메모리 단편화 방지. 연속적인 공간 사용 가능
    - 단점
        - 메모리 낭비 : 메모리의 절반을 비워두는 구조이므로 비효율적으로 사용할 수 있음
        - 큰 객체 처리 어려움 (복사시 오래걸림)

- G1 GC(Garbage First Garbage Collector) : 힙 영역을 여러 개의 작은 영역으로 나누고, 각 영역을 독립적으로 관리하는 방식. 각 영역에서 먼저 가비지가 많이 발생한 곳부터 수집

    - Region-based : 힙 영역을 Region으로 나누고, 각 Region에서 가비지가 많은 순서대로 수집
    - Concurrent Marking : 마킹을 동시에 수행하여, 스톱 더 월드 시간을 줄임
    - 장점
        - 예측 가능한 응답 시간
        - 효율적인 메모리 관리
        - 병렬 처리
    - 단점
        - 복잡성
        - 처리 오버헤드 (작은 Region 관리시)

- 요구사항에 따름

    - Mark-and-Sweep/ Compact : 단순하지만 스톱 더 월드가 길어질 수 있음
    - Generational GC : 대부분의 자바 애플리케이션에 적합, 객체 생애 주기에 따라 효율적
    - G1,ZGC,Shenandoah : 대규모 애플리케이션에서 지연 시간을 최소화하는데 적합

  ### Java 8 버전 추가

- 람다 표현식

    - 익명 클래스를 보완하기 위해서 만든 것

    - 인터페이스에 메소드가 **하나** 인 것들에 대해서만 적용 가능

    - ```java
    private void calculateClassic() {
    	Calculate calculateAdd = new Calculate() {
    	@Override // 익명클래스 사용
    	public int operation(int a, int b) {
    		return a+b;
    		}
    	};
    }
    
    // 람다 사용
    @FunctionalInterface
    private void calculateLambda() {
      Calcultae calculateAdd = (a,b) -> a+b;
    }
    ```

- 함수형 인터페이스

- 스트림

    - 연속된 정보를 처리하는데 사용

    - | filter                | 데이터를 조건으로 거를때 사용                                |
          | --------------------- | ------------------------------------------------------------ |
      | map                   | 데이터를 특정 데이터로 변환                                  |
      | forEach               | for 루프                                                     |
      | flatMap               | 변환된 데이터를  중첩된 스트립을 평탄화하여 하나의 스트림으로 변환 |
      | sorted                | 데이터 정렬                                                  |
      | toArray               | 배열로 변환                                                  |
      | any / all / noneMatch | 일치하는것 찾음                                              |
      | findFirst / Any       | 맨 처음이나 순서와 상관없는 것을 찾음                        |
      | reduce                | 결과를 취합                                                  |
      | collect               | 원하는 타입으로 데이터를 리턴                                |

- 옵셔널

    - orElse : 객체 값이 있든 없든 대체 값이 항상 계산됨
    - orElseGet : Supplier를 통해 제공되며 필요할때만 계산됨


#### 자바에 비해 코틀린이 가지는 이점

- 간결성(getter, setter, 생성자 등 수동 작성 필요 없음)
- null safety
- 확장함수 (기존 클래스를 수정하지 않고도 기능을 확장할 수 있음)
- 코루틴 - 비동기 프로그래밍을 쉽게 구현 가능
- 데이터 클래스
- 함수형 프로그래밍 지원(람다, 고차함수, 불변성)
- 코틀린에서는 기본형-참조형 타입 구분 없이 객체로 취급(내부에서 성능을 위해 기본형 타입 사용)


