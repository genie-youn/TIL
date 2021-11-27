# npm에 실행파일 배포하기

npm을 통해 실행파일을 배포하고 싶다면 `package.json`의 bin 필드에 선언하면 된다.

```json
{
  "bin": {
    "myapp": "./cli.js"
  }
}
```

을 선언하면 해당 패키지가 globally 하게 설치된다면 PATH에 `myapp`이 추가되어 `myapp`이라는 명령어를 사용하면 `cli.js`가 실행되게 된다.

즉 `myapp`을 설치하면 `cli.js`에 대한 심볼릭링크가 `/usr/local/bin/myapp`에 추가된다.

만약 실행파일이 하나뿐이라면 간단하게 실행파일의 경로만 문자열로 입력한다면 패키지의 이름이 명령어가 된다.

```json
{
  "name": "my-program",
  "version": "1.2.5",
  "bin": "./path/to/program"
}
```

이는 다음과 동일하다.

```json
{
  "name": "my-program",
  "version": "1.2.5",
  "bin": {
    "my-program": "./path/to/program"
  }
}
```

bin 필드에 참조되는 파일이 반드시 `#!/usr/bin/env node` 화인해야한다. 그렇지 않다면 노드의 실행없이 스크립트가 실행되게 된다.

> reference: https://docs.npmjs.com/cli/v7/configuring-npm/package-json#bin


