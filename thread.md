### 스레드

- 프로세스와 스레드

    - 프로세스 : 실행중인 프로그램. 각 프로세스는 독립적임

    - 스레드 : 프로세스 내에서 실행되는 작은 실행 단위. 하나의 프로세스는 여러개의 스레드를 가질 수 있음. 같은 메모리 공간을 공유하면서 작업 수행

        - 공유 되는 메모리 : 힙 메모리(객체, 클래스 변수)

        - 공유되지 않는 메모리 : 스택 메모리(스레드별 지역 변수), ThreadLocal 변수를 사용한 데이터 => 각 스레드마다 독립적으로 존재

        - ThreadLocal

            - 각 스레드마다 별도의 내부 저장소를 제공하여 해당 스레드에서만 접근할 수 있는 특별한 저장소

            - ```java
        public class Service {
          private ThreadLocal<String> cnt = new ThreadLocal<>(); // 변수 생성시 스레드 로컬 적용
        }
        ```

            - 같은 인스턴스 필드지만 스레드마다 다른 결과가 반환

            - 주의사항

                - 모두 사용하고 나면 remove()를 호출하여 스레드 로컬에 저장된 데이터를 반드시 제거해야 함
                - WAS처럼 스레드 풀을 사용하는 경우, 스레드 로컬에 데이터가 저장된 스레드를 스레드 풀에 반환하고, 이후 스레드 풀에서 동일한 스레드를 꺼내서 사용하면 이전에 저장한 스레드 로컬의 데이터가 반환되어 버그 발생 가능(스레드는 재사용되므로 데이터가 오염됨)

            - Excutor - 스레드 관리 및 비동기 작업 실행을 위한 도구. 스레드 풀을 통한 자원관리시 사용

    - Race Condition

        - 임계영역(critical section)이 두 개 이상의 스레드에 의해 동시에 실행되는 조건

        - 유형

            - Read - modify - write (읽기 수정 쓰기)

            - Check - then - act (확인 후 조치)

            - ```java
        if (map.contains(key)) {
        	map.remove(key); // 다른 스레드에서 이미 제거했다면 오류 발생 가능
        }
        ```

    - 해결 방법

        1. 상호배제(Mutal exclusion) -- 자바에서 채택

            - 스레드가 공유 변수 또는 공유 스레드를 사용하는 경우 다른 스레드가 동일한 작업을 수행하지 못하도록 배제

            - synchronized 키워드 사용 (메서드 앞 또는 메서드 내 블록처리)

                - 객체 잠금을 사용하여 해당 객체에 대한 동시 접근 제어
                - 느리고 무거움

            - volatile 키워드 사용

                - 변수에 사용 (여러 스레드 간에서 즉시 일관성 있게 보이도록 보장)
                - CPU 캐시가 아닌 메인 메모리에서 읽고 쓰도록 보장하여 동기화 함
                - 복합연산 등 원자성을 보장하지는 않음
                    - 단일 변수에 대해 읽고 쓰기가 빈번할 때
                    - 간단 플래그나 상태값 처럼 단순한 값의 변경에 대한 가시성을 보장할 때

            - Atomic 클래스

                - java.util.concurrent.atomic

                - 원자적 연산을 보장하는 클래스

                - Compare-And-Swap(CAS)알고리즘으 사용하여 경쟁상태 해결

                    - 값이 다른 스레드에 의해 변경되었는가 -> 않았다면 값을 수정

                - 스레드 safe한 원자적 연산이 필요할때

                - ```java
           // synchronized
           public synchronized void increment() {
             count++;
           }
           // synchronized-block
           public void increment() {
             synchronized(this) {
               count++;
             }
           }
           
           // volatile
           public class Counter {
             private volatile int count = 0;
             
             public void setCount(int count) {
               this.count = count;
             }
           }
           
           // Atomic
           public class Counter {
             private AmoticInteger count = new AtomicInteger(0);
             
             public void increment() {
               count.incrementAndGet(); // 원자적 증가
             }
           }
           ```



       - Semaphore : 자원에 대한 접근을 제한할 때

         - 특정 수의 스레드만 접근을 허용하는 상호배제 매커니즘

       - ReadWriteLock : 읽기 - 쓰기 작업을 분리

         - 여러 쓰레드가 동시에 읽기 작업 가능
         - 쓰기 작업에서는 모두 차단

    2. 프로세스 동기화 (Synchronize the process)

       - 한번에 하나의 프로세스만 공유데이터에 액세스

       - 파일 잠금 (File Locking)
       - 데이터베이스 잠금
       - 네트워크 소켓

       

    3. 상호배제와 프로세스 동기화 채택 방식

       1. 상호 배제 : 공유 자원에 대한 경쟁 상태 방지. 자원의 일관성을 보장
       2. 프로세스 동기화 : 프로세스 간의 순서를 제어하고 통신이나 작업 흐름을 동기화. 다른 프로세스의 작업이 완료된 후에만 실행되어야하는 작업이 있을 때
