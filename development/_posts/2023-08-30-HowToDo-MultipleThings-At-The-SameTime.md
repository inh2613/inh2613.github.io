---
layout: post
title: 내 컴퓨터는 어떻게 동시에 많은 일을 할까? (OS-2)
description: >
  내 컴퓨터는 어떻게 카톡도 보고 유투브도 보고 동시에 일을 할 수 있는지
sitemap: false
hide_last_modified: true
---

---

## Thread
- a lighit weight process
- 프로세스의 실행 단위
- thread id, program counter, register set, stack
- 같은 프로세스에 속한 다른 스레드와 code, data heap 영역 공유
- **register, stack, program counter**는 공유 ❌
<img width="810" alt="image" src="https://github.com/inh2613/inh2613.github.io/assets/62206617/a3fe34c1-682c-4c70-953b-7ef7a2bc2e48">

## ❗️ Process vs Thread
- Process
  - 실행하고 있는 프로그램
  - OS로부터 메모리 자원을 할당받아 CPU를 할당받아 실행되는 것
  - 메모리 자원은 code, data, heap, stack영역이 있음
  - 프로세스 간에 자원(메모리)공유 안함
- Thread
  - 한 프로세스의 실행 단위
  - 스레드 간 자원(메모리)공유함
  - 한 프로세스에서 code, data, heap 영역은 공유하지만 **stack영역**은 독립적으로 할당 받음

## OS에서의 `동시에`
  - ⭐️⭐️⭐️ **Concurrency**
    - CPU core가 1개 일 때 짧은 시간동안 왔다갔다 번갈아가면서 수행해서 time sharing 시스템으로 마치 **동시에** 실행되고 있는 것 처럼 보이는 것

  - Parallelism
    - 여러 CPU 코어로 각각 연산해서 **동시에** 실행하는 것

## Context Switch
- Context
  - 프로세스의 실행 상태를 나타내는 정보
  - register의 현재 값, Program counter, Stack pointer 등
  - PCB에 저장됨
- [Scheduling]()
- Context Switch
  - 실행중인 프로세스에서 새로운 프로세스로 CPU 제어권을 변환하는 과정
  - 실행중이던 프로세스의 PCB는 저장하고 새로운 프로세스의 PCB를 읽어서 state를 복구함

## Multi Programming & Multi Processing & Multi Threading
- Multi Programming
  - (1개의) CPU를 계속 일하게 하자
  - 프로세스 A가 I/O 작업을 수행하는 동안에는 CPU가 프로세스 B를 실행하도록 전환, 하나의 CPU에서 여러 프로세스가 시간을 나눠서 실행(Concurrency of time sharing)
- Multi Processing
  - 2개 이상의 process가 동시에 실행되는 것
  - 1개의 CPU
    - Concurrency -> time sharing -> context switch
  - 여러 core의 CPU
    - Parallel
    - 진짜 동시에 수행
- Multi Threading
  - 한 프로세스 내에서 여러개의 실행 단위인 스레드가 실행 되는 것
  - 마찬가지로 1개의 cpu에서 time sharing 시스템 을 이용해서 동시에 실행될 수 있고
  여러 core의 cpu로 병렬적으로 동시에 실행될 수 있다. 
  - cpu는 한번에 하나의 **스레드**만 실행할 수 있다. 

- Multi Process vs Multi Thread
  - Multi Process
    - 많은 메모리 공간 & CPU 시간 차지
    - 다른 process에 영향을 안줘서 안정성 높음
    - Context switching 느림
  - Multi Thread
    - 적은 메모리 공간 & CPU 시간 차지
    - 동기화 문제, 전체 스레드가 잘 못 될 위험이 있음
    - Contexst switching 빠름(캐시 메모리를 초기화할 필요가 없어서)
    - 동기화 문제
      - 서로 다른 thread는 stack영역을 제외하고 메모리영역을 공유하기 때문에 동일한 자원에 동시에 접근하여 잘못된 값을 읽거나 수정하는 경우
    - Heap 영역을 이용하여 다른 thread와 통신함

  
  - 메모리 구분이 필요할 때 -> Mult Process
  - Context switching이 빈번할 때 -> Multi Thread
  - 데이터 공유가 빈번하고 자원을 효율적으로 사용해야 할 때 -> Multi Thread
  - 통신(IPC)에 의한 오버헤드를 줄이고 싶을 때 -> Multi Thread

## 내 컴퓨터는 어떻게 카톡도 보고 유튜브도 보고 동시에 일을 할 수 있을 까?
- 카톡 메세지 보면서 유튜브를 볼 수 있음(앱들이 다를 때) -> 멀티 프로세싱
- 카톡을 사용하는데, 긴 동영상 메세지를 보내면서 다른 채팅방에서 메세지를 수신할 수 있음(하나의 앱에서) -> 멀티 스레딩

🤔 다음 포스팅은 **멀티 프로세스 & 스레드가 어떻게 서로 통신**하는지, **멀티 프로세싱 & 스레딩 상황에서 발생할 수 있는 문제**들에 대해 알아보자! 