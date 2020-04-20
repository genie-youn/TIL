# Github Action 으로 GKE 에 Spring Boot 애플리케이션 배포하기

## 들어가며
요즘 하고있는 사이드 프로젝트에 배포환경을 구성하면서 Github Action 을 처음 사용해보았다.
이제 간단한 프로젝트에 따로 jenkins 를 만들어 올릴 필요가 없겠구나 하는 감탄이 나올만큼 편리하고, 많은 integration 들을 준비해두어 놀라웠다.

그렇지만 내가 찾지 못한건지.. 단순히 따라하면 뚝딱 되는 Getting Start 가 없어, 약 세시간 가량 삽질을 했고 다음에 똑같은 삽질을 하지 않기위해
Github Action을 통해 GKE에 Spring Boot 애플리케이션을 배포하는 방법은 간단히 남겨두려 한다.

## 준비
우선 당연하게도 배포할 스프링 부트 애플리케이션이 필요하다. 간단히 spring initializer 를 통해 스프링 부트 애플리케이션을 만들었다.
> 예제의 코드는 다음 [레파지토리](https://github.com/genie-youn/deploy-gke-example) 를 참고한다.

또한 [GCP](https://cloud.google.com/gcp) 에도 프로젝트가 생성되어 있어야 한다.

그리고 로컬에 [Docker](https://docs.docker.com/) 가 설치되어 있다고 가정한다.

## Containerize
컨테이너에 올릴 애플리케이션 이미지를 Containerize 해보자.

### 1. Dockerfile 생성
프로젝트 루트에 `Dockerfile` 파일을 만들고 다음과 같이 수정한다.

```Dockerfile
FROM openjdk:8-jdk-alpine

# root 권한으로 실행되지 않도록 사용자 그룹 추가.
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

> gradle 을 사용할 경우를 기준으로 작성되었다. 만약 maven 을 사용중이라면 `ARG JAR_FILE=build/libs/*.jar` 대신 `ARG JAR_FILE=target/*.jar` 로 Dockerfile 을 생성하거나 빌드시 `--build-arg JAR_FILE=target/*.jar` 파라미터를 추가로 주도록 하자

> 자세한 내용은 [레퍼런스](https://spring.io/guides/gs/spring-boot-docker/) 를 참고

### 2. 실행
이미지가 정상적으로 생성되었는지 확인하기 위해 로컬에서 실행시켜보자.

```shell
$ ./gradlew build
$ docker build -t demo .
$ docker run -p 8080:8080 demo
```

### 레지스터에 업로드
[quickstart](https://cloud.google.com/container-registry/docs/quickstart)의 '시작하기 전에'' 를 따라서 Container Registry 를 활성화하고 Cloud SDK 를 설치한다.

설치를 마쳤으면 cloud sdk 를 통해 인증을 하고 이미지에 태그를 지정하고 업로드한다.

```shell
$ gcloud auth configure-docker
# [PROJECT-ID] 에는 gcp 프로젝트의 id를 넣어준다. 프로젝트 id가 test 라면 gcr.io/test/demo 로 지정하면 된다.
$ docker tag demo gcr.io/[PROJECT-ID]/demo
# 레지스트리에 푸쉬
$ docker push gcr.io/[PROJECT-ID]/demo
```

[Container Registry](https://console.cloud.google.com/gcr) 에 접속해서 정상적으로 업로드 되었는지 확인한다.

## Github Action

### 1. workflow 생성

상단에 네번째 `Actions` 클릭

![](https://user-images.githubusercontent.com/16642635/79749277-e1a8aa80-8349-11ea-850d-3bc225d14e81.png)

Build and Deploy to GKE > `Set up this workflow` 클릭

![](https://user-images.githubusercontent.com/16642635/79749526-54198a80-834a-11ea-8614-e972b7761c08.png)

### 2. workflow 정의
`Set up this workflow` 를 클릭하고 나면 기본적인 플로우가 정의되면 yaml 파일이 생성된다. 한 부분씩 살펴보도록 하자.

```yaml
on:
  release:
    types: [created]
```

github에 어떤 액션이 생겼을 때 workflow 를 실행시킬 것인지 정의하는 부분이다.

몇가지 유용하게 사용할 만한 액션들은 다음과 같다.

```yaml
on:
  # 릴리즈 생성시
  release:
    types: [created]
  # develop 브랜치에 푸쉬 발생시
  push:
    branches:
      - develop
  # feature 이하 모든 브랜치, develop, master 브랜치에 PR이 생성됐을 경우
  pull_request:
    branches: [feature/**, develop, master]
  # 이슈 (PR을 포함하여)에 코멘트가 생성되었을 경우
  issue_comment:
    types: [created]
```
