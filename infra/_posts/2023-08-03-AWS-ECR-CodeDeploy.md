---
layout: post
title: AWS ECR & CodeDeploy
description: >
  Docker, ECR, CodeDeploy를 이용한 CI/CD 구축 과정
sitemap: false
hide_last_modified: true

---

---

## 목차

[구조](#구조)

[IAM 역할 생성](#IAM-역할-생성)

[CodeDeploy Agent 설치](#codedeploy-agent-설치)

[ECR 생성](#ecr-생성)

[CodeDeploy 생성](#codedeploy-생성)

[S3 생성](#s3-생성)

[Dockerfile](#dockerfile)

[appspec.yml](#appspecyml)

[Github Actions](#github-actions)

[결과](#결과)

[장점](#장점)

[참고 자료](#참고자료)


## 구조
![ci-cd-test drawio](https://github.com/inh2613/inh2613.github.io/assets/62206617/44f81604-ca8b-46f6-8c19-23cdb6925bc2)

## IAM 역할 생성

1. EC2
![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/f103e666-938e-432b-89aa-c3f9280eaef0)
EC2 콘솔에 들어가서 EC2에게 IAM 역할을 부여한다.
![11](https://github.com/inh2613/inh2613.github.io/assets/62206617/4ca32009-ba39-48e9-908d-17e3b84af57e)

2. CodeDeploy
![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/c1157256-0a8a-4f96-8527-b2e6b6c0e1be)

## CodeDeploy Agent 설치

EC2에 [code-deploy-agent](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/codedeploy-agent-operations-install-ubuntu.html) 설치

❗️ 권한 설정이 잘 못 된 채로 agent를 설치했다면(agent 설치 후 console에서 IAM 정책을 변경했다면) EC2에서 권한 관련한 파일을 삭제하고 agent를 restart하는 과정이 필요하다.

```sh
# AWS 자격증명 파일 삭제
$ sudo rm -rf /root/.aws/credentials

# codedeploy-agent 재시작
$ sudo systemctl restart codedeploy-agent
```

## ECR 생성
돈을 지불하고 싶지 않아 Public Registry로 생성했다. 

❗️ Public으로 repository를 생성하니 리전이 `us-east-1`로 생성되었다(private은 서울 리전으로 잘 생성됨).

github actions에서 ECR에 로그인 하는 과정에서 리전 헷갈리면 안된다.

## CodeDeploy 생성

1. 애플리케이션 생성

![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/9a753fff-c7ce-4d5b-a7f2-8f56e2ab9f44)
2. 배포그룹 생성

![11](https://github.com/inh2613/inh2613.github.io/assets/62206617/85972386-1bc3-4189-9709-053db4a0b7e1)

이때 앞서 IAM 역할 생성할 때 만든 code-deploy-role를 서비스 역할에 입력한다. 

## S3 생성
모든 퍼블릭 액세스는 차단했다.


## Dockerfile
프로젝트 최상단에 Dockerfile를 작성한다.

```docker
FROM openjdk:17 AS builder
COPY gradlew .
COPY gradle gradle
COPY build.gradle .
COPY settings.gradle .
COPY src src
RUN chmod +x ./gradlew
RUN microdnf install findutils
RUN ./gradlew build -x test --no-daemon


FROM openjdk:17-alpine
RUN mkdir /opt/app
COPY --from=builder build/libs/gifthub-0.0.1-SNAPSHOT.jar /opt/app/spring-boot-application.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar","/opt/app/spring-boot-application.jar"]
```
이때 `xargs is not available` error가 발생해서 아래 명령을 추가했다. gradle 7.5 버전에서 발생한다고 한다.


```docker
RUN microdnf install findutils
```

이 Dockerfile은 이미지 빌드 시 실행될 것이다. 또한 이미지 용량을 줄이는 다양한 최적화 방법이 시도될 수 있을 것이다.

또한 EC2에 미리 docker가 [설치](https://docs.docker.com/engine/install/ubuntu/), [권한 설정](https://technote.kr/369)이 되어 있어야 한다.


## appspec.yml

프로젝트 최상단에 작성한다.

codedeploy를 사용하기 위해 필요하며, 후에 github actions에 동적으로 작성되는 `deploy.sh`와 함께 S3에 zip파일로 저장될 것이다.

```yml
version: 0.0
os: linux
files:
  - source: /
    destination: /home/ubuntu/app
    overwrite: yes

permissions:
    - object: /
      pattern: "**"
      owner: ubuntu
      group: ubuntu
      mode: 755

hooks:
  AfterInstall:
    - location: scripts/deploy.sh
      timeout: 1800
      runas: ubuntu
```

## Github Actions

```yml
name: ci/cd

on:
  push:
    branches: [ "dev" ]

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Pull Request
        uses: actions/checkout@v3
        with:
          ref: ${{ github.event.pull_request.head.ref }}

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-northeast-2

      - name: Login to public ECR
        uses: docker/login-action@v2
        with:
          registry: public.ecr.aws
          username: ${{ secrets.AWS_ACCESS_KEY_ID }}
          password: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        env:
          AWS_REGION: us-east-1

      - name: check file path
        run: |
          ls -al
          pwd
        shell: bash

      - name: build and push image to AWS ECR
        env:
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t ${{secrets.AWS_PRIVATE_ECR_REGISTRY}}/ci-cd-test:$IMAGE_TAG .
          docker push ${{secrets.AWS_PRIVATE_ECR_REGISTRY}}/ci-cd-test:$IMAGE_TAG
          
          mkdir scripts
          touch scripts/deploy.sh
          echo "aws ecr get-login-password --region us-east-1  | docker login --username AWS --password-stdin ${{secrets.AWS_PRIVATE_ECR_REGISTRY}}" >> scripts/deploy.sh
          echo "docker pull ${{secrets.AWS_PRIVATE_ECR_REGISTRY}}/ci-cd-test:$IMAGE_TAG" >> scripts/deploy.sh
          echo "docker run -p 8080:8080 -d --restart always --name ci-cd-test ${{secrets.AWS_PRIVATE_ECR_REGISTRY}}/ci-cd-test:$IMAGE_TAG" >> scripts/deploy.sh

      - name: upload to s3 with appspec.yml and deploy.sh
        env:
          IMAGE_TAG: ${{ github.sha }}
        run: |
          zip -r deploy-$IMAGE_TAG.zip ./scripts appspec.yml
          aws s3 cp --region ap-northeast-2 --acl private ./deploy-$IMAGE_TAG.zip s3://inhee-ci-cd-test

      - name: deploy to EC2 server
        env:

          IMAGE_TAG: ${{ github.sha }}
        run: |
          aws deploy create-deployment --application-name ci-cd-test \
                --deployment-config-name CodeDeployDefault.OneAtATime \
                --deployment-group-name ci-cd-test \
                --s3-location bucket=inhee-ci-cd-test,bundleType=zip,key=deploy-$IMAGE_TAG.zip
                
```

1. github 레포의 최신상태를 가져온다.
2. java 17 버전 setup
3. AWS 로그인 (IAM에서 보안자격증명에 CSV파일 받을 수 있음)
4. ECR 로그인 (private ECR인지 public ECR인지 로그인 방법이 다르므로 [자료](https://github.com/marketplace/actions/docker-login#aws-elastic-container-registry-ecr) 참고하기)
5. 파일 확인
6. docker 이미지 빌드 및 ECR에 이미지 push
    - 이미지 별로 버전을 구분할 수 있도록 `github.sha` 값으로 이미지 태그 값을 설정함
    - 이미지가 계속 바뀌므로 `deploy.sh` 를 동적으로 작성함
7. `deploy.sh`와 `appspec.yml`을 S3에 업로드
8. CodeDeploy 실행
    - S3에서 deploy.sh와 appspec.yml을 가져와 배포

## 결과

1. Github Actions

![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/34bf50f2-6b16-4833-8cc9-a3d8d0143b93)

2. ECR

![11](https://github.com/inh2613/inh2613.github.io/assets/62206617/27b4b24f-19ff-46b8-8031-59dc6b3a505f)

3. S3

![11](https://github.com/inh2613/inh2613.github.io/assets/62206617/215730d3-b87b-4761-97ef-03b33804e82f)

4. CodeDeploy

![11](https://github.com/inh2613/inh2613.github.io/assets/62206617/dd60a712-d231-457a-b210-2387044b34ae)

5. EC2

![11](https://github.com/inh2613/inh2613.github.io/assets/62206617/766ba248-13fc-4eb3-867e-5356f273b716)

## 장점

복잡하긴하다. 그냥 ssh tunnelling이용해서 docker image pull 해서 배포하는게 가장 쉬운 것 같다.

그래도 장점이 있다면 배포 시간이나 어떤 버전을 배포했는지 모두 기록되기 때문에 배포 버전관리가 쉽다는 장점이 있는 것 같다.

또한 나중에 무중단 배포 할때, AWS CodeDeploy에서 Console로 쉽게 제공하기 때문에 이 부분에서도 이점이 있는 것 같다.

## 참고자료

- [Github Actions + ECR + Auto Scaling Group + EC2 + CodeDeploy + S3 를 사용하여 Blue/Green CI/CD 구축하기](https://velog.io/@kshired/Github-Actions-ECR-Auto-Scaling-Group-EC2-CodeDeploy-S3-%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EC%97%AC-BlueGreen-CICD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0#codedeploy-auto-scaling-policy)