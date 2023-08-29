---
layout: post
title: 프로그램은 어떻게 실행될까 (OS-1)
description: >
  내 노트북에서 프로그램은 어떻게 실행될까? 그리고 프로세스는 도대체 뭘까?
sitemap: false
hide_last_modified: true
---

---
## OS
- OS is a **program** running at all times on the computher

## [Memory hierarchy](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=techref&logNo=222246966805)
![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/7f1a50f9-7db2-4c05-aa06-213a8b8e12e6)
- Register
  - CPU내부에 있는 매우 작고 빠른 저장 공간
  - **지금 하고 있는 연산에 필요한 데이터**들만 저장
  - RAM에서 데이터 로드해서 Register에서 연산하는 것임
- Cache
  - 보통 SRAM(비쌈)
- RAM
  - 보통 DRAM
  - Memory영역으로 구성
- 주기억 장치
  - RAM, ROM, cache memory
- 보조기억 장치
  - HDD, SSD 등

## Memory 영역(RAM에만 있음)
- Code section
  - 우리가 작성한 소스코드가 들어가는 부분
  - 즉, 실행할 프로그램의코드가 저장되는 영역
- Data section
  - 전역변수, static 변수
- Heap section
  - 동적할당
  - 프로그래머가 할당/해제 하는 메모리 공간
  - 런타임 시에 크기 결정
  - java의 [garbage collector](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)
- Stack section
  - 프로그램이 자동으로 사용하는 임시 메모리 영역
  - 함수 호출시 생성되고 호출 끝나면 사라짐
  - Heap과 Stack 영역은 서로 trade off

## 그래서 프로그램은 어떻게 실행될까? 
![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/6f759aee-e885-4419-8225-17bbc422dab8)
1. 프로그램 실행 요청
2. HDD or SSD같은 보조 저장장치에 저장된 프로그램을 메모리(RAM)에 로드
3. RAM에서 CPU로
  - 직접 RAM에서 명령어를 처리하지 않고, CPU내부의 레지스터로 이동시킨다.
4. 레지스터에서 명령어를 실행하고 결과를 RAM에 저장하며 메모리 상태를 업데이트 한다.
  - 동적 메모리가 할당 되면 FreeStore 영역을 사용한다.(아래쪽으로 이동)
  - 스택 메모리가 할당되면 FreeStore영역을 사용한다. (위쪽으로 이동)

❗️ `free store`는 보통 C++에서 사용되는 개념이고 java에서는 `method` 영역이 비슷한 역할을 한다고 한다.
가장 중요한건 stack, heap 이다~

❗️ `stack`, `heap` 영역은 RAM의 영역이다  

## Process란?
- **실행중인 프로그램**
- **독립적인 메모리 자원 할당**
- 운영체제로부터 자원을 할당받는 작업의 단위
- HDD에 있는 program(명령어의 집합)이 memory 영역에 올라온 것
- 각각 독립된 메모리 영역(Code, Data, Heap, Stack)을 할당 받음
- 시스템 프로세스(kernel process), 사용자 프로세스로 나뉨
- 기본적으로 최소 1개의 스레드를 가지고 있음
- 한 프로세스는 다른 프로세스의 변수나 자료구조에 접근할 수 없음

## Process Status
- new
- ready
- running
- waiting
- terminated

![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/7fbcc832-fbad-43c2-84f9-60818ff56f6a)

## PCB(Process Control Block)
- 프로세스의 상태와 관련된 정보를 저장하는 데이터 구조
- 프로세스의 생성과 동시에 고유한 PCB 생성
- RAM에 저장됨
- 구성 요소
  - Process ID : 프로세스 식별 번호
  - Process status
  - Program counter : 프로세스가 다음에 실행할 명령어의 주소
  - CPU Register : 프로세스의 현재 상태에 관한 정보 포함, Context swicthing 시 저장하거나 복원해야 하는 레지스터들의 값
  - CPU 스케줄링 정보 : 프로세스의 우선순위, 스케줄 큐에 대한 포인터 등
  - Memory 관리 정보 : 페이지 테이블 또는 세그먼트 테이블 등과 같은 정보 포함
  - I/O 상태 정보 : 프로세스에 할당된 입출력 장치들과 열린 파일 목록
  - Accounting 정보 : 사용된 CPU 시간, 시간제한, 계정 정보 등

🤔 다음 포스팅은 **스레드**와 내 컴퓨터는 어떻게 카톡도 보고 유투브도 보고 **동시에 일을 할 수 있는지**에 대해 알아보자
