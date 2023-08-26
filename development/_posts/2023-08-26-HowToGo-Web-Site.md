---
layout: post
title: Network 정리
description: >
  내 노트북에서 네이버 홈페이지가 열리는 과정
sitemap: false
hide_last_modified: true
---

---
# LAN 구간(ARP Request가 미치는 영역)
IP를 할당 받는 과정(DHCP->ARP)->도메인 IP 쿼리(UDP->DNS)->실제 연결(TCP)->데이터 전송(HTTP)
## 1. Wifi access point (a.k.a 공유기)
- 일종의 무선 라우터
- 유선 네트워크와 무선 네트워크 간의 브리지 역할
## 2. DHCP
### 호스트의 IP 주소와 각종 TCP/IP 프로토콜의 기본 설정을 클라이언트에게 자동으로 제공해주는 프로토콜
1. 노트북이 wifi access point와 연결됨
2. DHCP를 통해 사용가능한 ip주소 및 관련 네트워크 구성 정보를 받게 됨
![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/1b2c7ba7-dd8b-4799-a700-72568f1d06e2)

## 3. ARP
### IP주소를 물리적 네트워크 주소로(NIC의 MAC 주소) 대응 시키는 프로토콜(IP로 MAC얻기)
DHCP로 받은 IP주소를 사용해 통신하려면 이 IP주소의 물리적인 MAC주소가 필요함 
ARP를 통해 매칭되는 MAC 주소 알아냄

<span style="color:green"> 내 노트북 통신 준비 완료! </span>

---
# WAN 구간
### 1. 통신사
### 2. DNS
내 노트북은 www.naver.com에 대해서 물어봄 아래와 같은 순서로 요청함
DNS 작동 순서
1. 로컬 DNS 서버 -> 요청한 서버: 내 노트북
- 통신사 기지국 DNS 서버
- 캐싱되어 있으면 바로 내 노트북한테 ip 전달
- 없으면 2.으로 진행
2. Root DNS 서버 -> 요청한 서버: 로컬 DNS 
- TLD DNS을 관리하는 최상위 서버
- www.naver.com는 모르고 com은 안다 -> **TLD로 가봐**
3. TLD 서버 -> 요청한 서버: 로컬 DNS
- com, co.kr 같은 도메인을 관리하는 서버
- www.naver.com은 모르고 naver.com은 안다 -> **Authoritative DNS 서버로 가봐**
4. Authoritative 서버 -> 요청한 서버: 로컬 DNS
- 실제 개인 도메인과 IP 주소의 관계가 기록되는 서버
- www.naver.com 알아 -> **여기로 가라**
5. 로컬 DNS 서버
- www.naver.com ip 주소 캐시 저장 및 내 노트북한테 전달

 코드: `nslookup www.naver.com`

<span style="color:green"> 내 노트북이 www.naver.com IP주소 알아냄! </span>

### 3. TCP Connection setup
- 3 way hand shake
  - 목적 : 정확한 정보 전송을 위해 상대방을 확인하기 위함
  - 과정
  ![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/4280d8a7-9738-45d0-a33f-6dd8f6f82ada)

- [TCPvsUDP](https://cocoon1787.tistory.com/757)

### 4. HTTP/HTTPS (TCP Data transfer)
1. HTTP
HypterText Transfer Protocol의 약자로 서버-클라이언트 모델을 따르면서 request/response 구조로 웹 상에서 정보를 주고받을 수 있는 프로토콜
- Connectionless & Stateless

  - 서버 연결 후 요청에 응답을 받으면 연결을 끊어버림
  클라이언트의 이전 상태를 모름
  - 이를 위해 cookie, session,jwt 등장

- [HTTP Request Message and Response Message](https://velog.io/@gparkkii/HTTPMessage)
- [HTTP Method](https://inpa.tistory.com/entry/WEB-%F0%9F%8C%90-HTTP-%EB%A9%94%EC%84%9C%EB%93%9C-%EC%A2%85%EB%A5%98-%ED%86%B5%EC%8B%A0-%EA%B3%BC%EC%A0%95-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)
- Path Variable VS Query String
  - Path Variable
    구체적인 리소스를 식별하는데 사용
  -  Query String
    리소스들을 정렬, 필터링 혹은 페이징 하는 곳에 사용
- GET VS POST
  - GET
    - 서버에게 리소스 요청
    - 브라우저에 캐싱 가능
  - POST
    - 서버에게 데이터 처리 요청
    - 캐싱 불가
- PUT VS PATCH
  - PUT
    - 모든 리소스를 수정(멱등성)
  - PATCH
    - 일부 리소스만 수정

- [HTTP Status Code](https://inh2613.github.io/development/2023-06-18-HttpStatusCode/)
- [cookie/session/jwt]()

2. HTTPS
- HTTP는 탈취당하면 body가 다 보임 -> HTTPS 등장
- [SSL 인증과정](https://goodgid.github.io/TLS-SSL/#%EB%8C%80%EC%B9%AD%ED%82%A4)
  ![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/2f7909a5-4814-46d2-b33e-a0cb37755471)

<span style="color:green"> HTTP 요청과 응답을 받으면 그 응답을 렌더링해서 보여줌 내 노트북에 네이버 홈페이지가 나타남! </span>

<span style="color:pink"> HTTP 1.1에서는 keep alive를 지원해서 한번 요청 응답했다고 tcp 연결이 termination되지 않음 </span>

### 5. TCP Connection termination
- 4 way hand shake
  - 목적: 한번 더 확인하고 끝내기
  - 과정
  ![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/2f5aa650-cf5f-482b-bb4a-a93a9e44a8f1)


