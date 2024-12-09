#TIL
#TIL


<img width="1204" alt="Pasted Graphic 3" src="https://github.com/user-attachments/assets/54b5471e-81aa-48da-b7d6-9d739c7d7aa7">
<img width="1091" alt="image" src="https://github.com/user-attachments/assets/f7144707-7fcd-4706-bcf5-ae70b3732871">
<img width="483" alt="image" src="https://github.com/user-attachments/assets/b1636890-090e-48d5-8b6f-6c6566c47261">
<img width="675" alt="image" src="https://github.com/user-attachments/assets/bd3e34cc-0e1b-49e0-948a-4b58a06c9fab">

### GC 방식5가지
1. Serial Collector
  : 영 영역과 올드 영역이 연속적으로처리. 하나의 CPU 사용
```
    살아 있는 객체-> 에덴
    에딴 꽉차면 To Survivor 영역. 너무 크면 바로 old. From Survivor -> ToSurvivor
    To Survivor 꽉차면 에덴, 프롬은 올드로 이동
    올드 영역 : Mark-sweep-compact 사용
   
```
2. Parallel Collector
    : 다른 CPU가 대기 상태로 남아 있는 것 최소화. 영영역 병렬처리. 많은 CPU, 처리랑도 많음
   ```
    Old :  Mark-sweep-compact 사용
   ```
3. Parallel Compacting Collector
  : JDK 6부터  사용
Old 영역에서 표시 -> 종합(살아있는 객체의 위치 조사) -> 컴팩션 (스윕은 단일스레드가 올드 영역전체를 흝지만, 종합은 여러 그래드가 흝음) 
4. Concurrent Mark-Sweep (CMS) Collector
   : 로우 레이턴시 컬렉터. 힙메모리 크기가 클때 적합
   ```
   영영역 : 병렬 컬렉터와 동일
   올드 영역 : 초기 표시 -> 컨커런트(서버 수행하면서 살아 있는 객체표시) -> 재표시 (실행중 변경된 객체 다시 표기) -> 컨커런트 스윕 (표시된 쓰레기 정리)
    컴팩션 X. 빈공간 발생함
   ```
5. Garbage First Collector (G1)
   : 힙영역을 region으로 구성(약 2000개정도)

```
  영영역 : 
  1. 몇개의 구역을 영영역으로 지정
  2. 영이 꽉차면 GC 수행
  3. 살아있는것만 서바이벌 -> 새로운 서바이벌 영역이 되고, 계속 쌓다가 aging 즉 old영역으로 승력

  올드 영역:
  1. 초기 표시(STW) : Old 영역에 객체 중 서비이벌 영역 참조하는 객체 표시
  2. 기본 구역 스캔(해당 서바이벌 영역 흝음)
  3. 컨커런트 표시 : 전체 힙 영역에 살아 있는 객체 찾음 - 영 gc 발생하면 STW
  4. 재표시 : 힙에 살아 있는 객체 표시 작업 완료
  5. 청소 : 필요 없는 객체 지우고 , 비어있는 영역 초기화
  6. 복사 STW) : 살아있는 객체 빈 구역으로 모음
```
System.gc()는 성능에서 문제가 됨
zgc
<img width="907" alt="image" src="https://github.com/user-attachments/assets/a2800c7d-78bf-44a1-8cff-d457fe4d8054">

[visual GC]
- 명령어
jps : JVM 목록 보여줌
jstat : 상황 확인하는 명령어 (-gcutil, -gccapacilty) 서버의 GC 상황 확인 / 각 영역에 할당되어 있는 메모리의 크기를 KB 단위로 나타냄

[GC 튜닝을 해야할 시점] -- GC튜닝은 가장 마지막에 하는 작업
- JVM 메모리 크기 미지정
- 타임아웃 계속 발생
  목적
  - old 영역으로 넘어가는 객체의 수 최소화
  - Full GC 시간 줄이기
 
    메모리가 크면 GC횟수 감소, GC 수행시간 김

[JMX]
- Java Management Extensions (자바 기반 모든 애프리케이션을 모니터링하기 위해 만든 기술)
    1. 인스트루먼테이션 레벨 : 관리 빈즈 제공. 리소스들의 정보를 취합하여 에이전드로 전달
    2. 에이전트 레벨 : 리소스를 관리하는 역할 수행
    3. 분산서비스 레벨: 관리자를 구현하기위한 인터페이스와 컴포넌트 제공
 
[Visual VM]
- JDK bin 에 jconsole, jvisualvm 툴 있음 (콘솔 - 구식, 비주얼 - 신식)

    

프로파일러
<img width="567" alt="image" src="https://github.com/user-attachments/assets/fb916db0-e1ee-40d8-bda1-d9abe15da548">
<img width="363" alt="image" src="https://github.com/user-attachments/assets/707298d9-386c-4dea-be2c-3f44146ab229">
1. VisualVM
특징:
무료로 제공되는 JVM 기본 프로파일링 도구.
실시간 CPU, 메모리 사용량, GC 활동 모니터링.
힙 덤프 분석 및 스레드 상태 확인 가능.
장점:
설정이 간단하고, JVM과 기본적으로 호환.
실시간 모니터링과 간단한 성능 문제 분석에 적합.
적합한 용도:
소규모 애플리케이션.
초기 문제 진단 및 힙 덤프 확인.
2. YourKit
특징:
상용 프로파일링 도구로, CPU와 메모리 프로파일링에 강력한 기능 제공.
메서드 실행 시간, 객체 할당 추적, I/O 분석 가능.
원격 프로파일링 지원.
장점:
직관적인 UI와 상세한 분석 데이터.
애플리케이션 병목 및 메모리 누수 탐지에 효과적.
적합한 용도:
대규모 애플리케이션의 정밀 분석.
원격 서버 애플리케이션의 성능 최적화.
3. JProfiler
특징:
CPU, 메모리, 데이터베이스 호출, 네트워크 I/O 등을 분석.
스레드 경쟁 및 병렬 처리 문제 추적.
JDBC 호출, NoSQL 쿼리 분석 기능 포함.
장점:
다양한 프로파일링 옵션과 세부 데이터 제공.
직관적인 결과 시각화.
적합한 용도:
복잡한 애플리케이션에서 CPU 및 데이터베이스 성능 분석.
동시성 문제 및 병목 현상 탐지.

5. IntelliJ IDEA Profiler
특징:
IntelliJ IDEA에 내장된 프로파일링 기능.
CPU, 메모리 사용량, 메서드 호출 빈도 분석.
GC 및 힙 상태 실시간 모니터링.
장점:
IDE와 통합되어 사용이 간편.
개발 중 성능 문제를 즉시 확인 가능.
적합한 용도:
코드 작성 중 성능 최적화.
빠른 문제 진단.

<img width="540" alt="image" src="https://github.com/user-attachments/assets/a0a270a3-845c-4b09-8d65-17fe22d59985">



