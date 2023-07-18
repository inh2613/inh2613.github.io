---
layout: post
title: AWS SubDomain
description: >
  서브도메인 생성 과정
sitemap: false
hide_last_modified: true
---

---
### 순서

1. ACM에서 하위 도메인 인증서 발급
    - `Route53에서 레코드 생성` 해야 DNS 검증 방식으로 생성됨
   ![Alt text](image.png)
    
    - Route53 레코드 화면
    ![Alt text](image-1.png)



2. A 레코드 생성
    ![Alt text](image-2.png)
   
3. 로드 밸런서에 인증서 등록
    ![Alt text](image-3.png)
    ![Alt text](image-4.png)