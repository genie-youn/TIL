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

#### Event

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
  # 이슈가 할당됐을 경우
  issues:
    types: [assigned]
```

> 사용가능한 전체 액션은 다음 [레퍼런스](https://help.github.com/en/actions/reference/events-that-trigger-workflows#webhook-events)를 참고한다.

#### Environment

다음은 `Environment` 이다. workflow 에서 사용할 환경 변수들을 정의할 수 있다.

또한 `Settings` > `Secrets` 에 등록해둔 private 한 정보들도 사용 가능하다.

![](https://user-images.githubusercontent.com/16642635/79864784-00c03e80-8415-11ea-8936-b6428ee3b394.png)

```yaml
env:
  GKE_PROJECT: ${{ secrets.GKE_PROJECT }}
  GKE_EMAIL: ${{ secrets.GKE_EMAIL }}
  GITHUB_SHA: ${{ github.sha }}
  GKE_ZONE: us-west1-a
  GKE_CLUSTER: example-gke-cluster
  IMAGE: gke-test
  REGISTRY_HOSTNAME: gcr.io
  DEPLOYMENT_NAME: gke-test
```

이후 `$GKE_CLUSTER` 와 같이 사용하면 된다.

```yaml
# Set up kustomize
- name: Build
  run: |        
    docker build -t "$REGISTRY_HOSTNAME"/"$GKE_PROJECT"/"$IMAGE":"$GITHUB_SHA" \
      --build-arg GITHUB_SHA="$GITHUB_SHA" \
      --build-arg GITHUB_REF="$GITHUB_REF" .
```

### Jobs

이후 실제 수행할 작업들의 목록인 `Jobs` 를 기술한다. 한단계씩 살펴보자

```yaml
jobs:
  # 빌드 후 gke에 배포
  setup-build-publish-deploy:
    name: Setup, Build, Publish, and Deploy
    runs-on: ubuntu-latest
    # 순차적으로 실행되는 작업들의 집합인 steps
    steps:
    - name: Checkout
      # {owner}/{repo}@{ref} 로 사용할 수 있다.
      # 이 스텝의 경우 https://github.com/actions/checkout 의 v2를 사용하게 된다.
      uses: actions/checkout@v2

    # gcloud CLI 설치
    - uses: GoogleCloudPlatform/github-actions/setup-gcloud@master
      with:
        version: '270.0.0'
        service_account_email: ${{ secrets.GKE_EMAIL }}
        service_account_key: ${{ secrets.GKE_KEY }}

    # gcloud command-line tool 을 사용하여 사용자 인증
    - run: |
        # Set up docker to authenticate
        # via gcloud command-line tool.
        gcloud auth configure-docker

    # Build the Docker image
    - name: Build
      run: |        
        docker build -t "$REGISTRY_HOSTNAME"/"$GKE_PROJECT"/"$IMAGE":"$GITHUB_SHA" \
          --build-arg GITHUB_SHA="$GITHUB_SHA" \
          --build-arg GITHUB_REF="$GITHUB_REF" .

    # Push the Docker image to Google Container Registry
    - name: Publish
      run: |
        docker push $REGISTRY_HOSTNAME/$GKE_PROJECT/$IMAGE:$GITHUB_SHA

    # Set up kustomize
    - name: Set up Kustomize
      run: |
        curl -o kustomize --location https://github.com/kubernetes-sigs/kustomize/releases/download/v3.1.0/kustomize_3.1.0_linux_amd64
        chmod u+x ./kustomize

    # Deploy the Docker image to the GKE cluster
    - name: Deploy
      run: |
        gcloud container clusters get-credentials $GKE_CLUSTER --zone $GKE_ZONE --project $GKE_PROJECT
        ./kustomize edit set image $REGISTRY_HOSTNAME/$GKE_PROJECT/$IMAGE:${GITHUB_SHA}
        ./kustomize build . | kubectl apply -f -
        kubectl rollout status deployment/$DEPLOYMENT_NAME
        kubectl get services -o wide
```
