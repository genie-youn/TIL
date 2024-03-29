cli
- bin/vue.js // 엔트리포인트 
  - 노드 버전 체크
    - 현재 process의 버전이 package.json.engines.node 에 명시된 버전을 만족시키는지 확인
    - 만족하지 않는다면 종료
  - `/packages/test` 가 path에 존재하고 상위 디렉토리에 `@vue`가 존재한다면 process.env.VUE_CLI_DEBUG = true 로 설정
  - create 명령어 입력시 lib/create.js 실행
    - create 호출
      - options.proxy가 존재하면 process.env.HTTP_PROXY = options.proxy 를 설정
      - cwd 에 현재 디렉토리 정보 할당
      - inCurrent 에 projectName이 "." 인지 할당
      - name 은 inCurrent 라면 현재 디렉토리의 이름 아니라면 projectName으로 할당
      - targetDir 할당
      - validateProjectName(name) // npm 패키지의 name으로 사용가능한 이름인지
      - validate-npm-package-name의 결과에 따라 console에 로그
      - targetDir 에 디렉토리가 이미 존재하고 options.merge가 아니라면
        - options.force 옵션이 존재하면
          - 해당 디렉토리 삭제 
        - 아니라면
          - 콘솔을 비운다음
          - inCurrent 라면
            - inquirer를 통해 현재 디렉토리에 프로젝트를 생성할것인지 확인하고 사용자가 ok 하지 않는다면 중단
          - 아니라면
            - inquirer를 통해 merge할지 overwrite 할지 취소할지 묻고 overwrite 라면 targetDir를 삭제 merge라면 계속 진행
      - targetDir 가 존재하지 않거나 options.merge 라면
        - @vue/lib/util/createTool.getPromptModules() 호출
          -  @vue/lib/promptModules 하위 모듈들을 import 한 뒤 이를 배열에 담아 반환
        - Creator 객체 생성 new Creator(name, targetDir, getPromptModules()) 
          - 이름 할당
          - context에 targetDir 할당, 기본값 process.env.VUE_CLI_CONTEXT
          - resolveIntroPrompts() 의 결과로 presetPrompt와 featurePrompt 할당
            - getPresets() 호출
              - @vue/lib/options의 loadOptions 호출
                - 캐시되어 있는 옵션이 잇다면 이를 반환
                - .vuerc 파일이 있는지 확인하고 없다면 빈값 반환
                - .vuerc 파일을 읽어서 캐시하고
                - 스키마를 검사한 다음 읽은 값을 반환
              - 반환받은 options의 presets과 기본값 presets merge하여 반환 
            - 저장된 프리셋의 이름과 features로 name을 구성하고 value는 preset인 presetChoices를 할당
            - presetChoices를 가지고 presetPrompt 생성
            - featurePrompt 생성
          - resolveOutroPrompts()의 결과로 outroPrompt 할당
            - config 파일 쓸지 물어봄
            - 저장할지 물어봄
            - 패키지매니저 뭐 사용할지 물어봄
          - this.run 에 this를 명시적으로 바인딩
          - PromptModuleAPI 인스턴스 생성
            - creator 프로퍼티에 자신을 생성한 Creator 할당
          - PromptModuleAPI 인스턴스를 파라미터로 promptModules 의 모든 모듈을 실행시킴
          - 각 모듈은 PromptModuleAPI를 파라미터로 받아 이 객체를 통해 Creator에게 메세지를 전달 할 수 있다.
            - 전달 가능한 메세지는 `injectFeature` `injectPrompt` `injectOptionForPrompt` `injectOptionForPrompt` 
        - Creator의 create(options) 호출
          - isTestOrDebug = process.env.VUE_CLI_TEST || process.env.VUE_CLI_DEBUG  
          - { run, name, context, afterInvokeCbs, afterAnyInvokeCbs } = this
          - preset이 존재하지 않으면
            - cliOptions에 프리셋이 존재하면 이를 가지고 resolvePreset
              - ㅁ
            - cliOptions에 프리셋이 없지만 cliOptions에 default가 있으면 기본 프리셋
            - cliOptions에 inlinePreset이 있으면 이걸 사용
            - 그게 아니라면 prompthAndResolvePreset()
              - ㅁ
          - 수정하기 전에 깊은 복사를 해서 원본을 수정하지 않게 하고
          - preset.plugins['@vue/cli-service'] 에는 프로젝트 이름과 프리셋을 머지하여 할당
          - cliOptions.bare가 존재하면 preset.plugins['@vue/cli-service'].bare 에 true 할당
          - preset.router가 존재하면 preset.plugins['@vue/cli-plugin-router'] 에 빈 객체를 할당하고 routerHistoryMode를 사용한다면 flag를 true로 선언 // 레거시를 위함
          - preset.vuex가 존재하면 preset.plugins['@vue/cli-plugin-vuex']에 빈 객체를 할당 마찬가지로 레거시를 위함
          - 패키지매니저를 결정하고
          - 콘솔을 초기화 한 다음
          - PackageManager 인스턴스 생성
            - ㅁ
          - 'creation' - 'creating' 이벤트 발생
          -  
    - Promise를 반환하게 되는데, 실패하면 스피너를 멈추고 테스트 에러로그를 남긴 후 process.env.VUE_CLI_TEST 가 아니라면 프로세스를 종료


vue ui
lib/ui ui()
- host 지정
- port 지정
- process.env.VUE_APP_CLI_UI_URL 초기화
- 기존 process.env.NODE_ENV는 백업하고 process.env.NODE_ENV 를 'production' 으로 엎어치기
- options.dev 가 있으면 process.env.VUE_APP_CLI_UI_DEBUG = true
- process.env.VUE_CLI_IPC 가 존재하지 않으면 process.env.VUE_CLI_IPC = `vue-cli-${shortid()}`
- options.quiet가 없으면 시작 로그
- opts 객체 정의
  - 그래프큐엘에 관련된 기본 설정들..
- httpServer = server()
  - a


@vue/cli-ui/server -> @vue/cli-ui/graphql-server
(options, callback)
- 파라미터로 받은 옵션에 {integratedEngine: false} 추가
- express 구동 후
- 옵션으로 함께 주입받은 @vue/cli-ui/apollo-server 패키지 하위 모듈들에서 typeDefs, resolvers, context, schemDirectives, pubsub 로드.. // 왜..? 어차피 같은 패키지안에 있는데 vue-cli-plugin-apollo/graphql-server 대신 이걸 사용하도록 변경하였다. cors 이슈가 있었나본데.. https://github.com/Akryum/vue-cli-plugin-apollo/blob/master/graphql-server/index.js 이걸 그대로 긁어왔네
- dataSource도 옵션으로 받은거 붙이고
- 스키마 만들고
- 아폴로 서버 띄우고.. 아무런 비즈니스 로직이 없다.

@vue/cli-ui/apollo-server/server.js
- 기본적으로 cli-ui/src 빌드 산출물이 dist로 떨어지고 이걸 우선 라우팅함.
- /public 은 asset들..
- /_plugin/:id 는 plugins.serve 에
- /_plugin-logo/:id 는 plugins.serveLogo에
- /_addon/:id 는 client-addons에

tasks
ProjectTasks
ProjectTaskDetails
그래프큐엘에 task로 요청해서 받아온걸 그대로 보여주고..
선택된 task를 가지고 ProjectTaskDetails
task 시작시 mutate {TASK_RUN, id}

@vue/cli-ui/apollo-server/connectors/task.js
run(id, context)



그래프큐엘이 웹소켓으로도 주고받을수 있구나..
