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
