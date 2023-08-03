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
   ![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/3bfe7fa4-e009-4f49-b22b-86f2319e3e0d)

    
    - Route53 레코드 화면
    ![image-1](https://github.com/inh2613/inh2613.github.io/assets/62206617/71defd1d-5aeb-4b8b-9529-fffaecf48beb)


2. A 레코드 생성
    ![image-2](https://github.com/inh2613/inh2613.github.io/assets/62206617/cd1b507b-5b63-4c33-b327-a95854562e1d)

   
3. 로드 밸런서에 인증서 등록
    ![image-3](https://github.com/inh2613/inh2613.github.io/assets/62206617/df9f5583-6706-49dc-8455-935b4636633f)

    ![image-4](https://github.com/inh2613/inh2613.github.io/assets/62206617/3a470f9b-967c-4405-9e99-6646c83feb39)
