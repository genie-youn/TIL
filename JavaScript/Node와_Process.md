# Node와 Process

### 들어가며
Node 환경에서 개발을 하다 보면  `process`란 객체를 쓸 일이 왕왕 생긴다. 특히 `process.NODE_ENV` 같은 아이들..
런타임 혹은 빌드타임에 무언과 실행환경과 관련된 값이 필요하면 보통 이 객체 안에 있으니 가져다 쓰겠는데, 이 아이는 왜 있는건지 언제부터 있던건지 누가 생성해주는것인지 궁금해져 정리하려 한다.

> 레퍼런스는 [다음](https://nodejs.org/api/process.html) 을 참고한다.

### Node에서의 Process
`process` 객체는 현재 Node.js 프로세스의 정보와 제어를 가능하게 하는 글로벌 객체이다. 모든 Node.js 어플리케이션이라면 접근 가능하다. (r명시적으로 `require`를 통해 접근할 지는 선택의 자유이다. 즉 없이도 접근 가능하다.)

한가지 신기한건 이 `process` 객체가 Node의 `EventEmitter` 타입의 인스턴스라는것.

다음과 같은 것들을 할 수 있다.

#### 이벤트 리스닝
프로세스의 이벤트를 감지하여 콜백을 실행할 수 있다.
```javascript
process.on('{hook}}', (code) => {
  console.log('do something: ', code);
});
```

감지할 수 있는 훅 (이벤트)은 다음과 같다.

- beforeExit: Node의 이벤트루프가 비어있고 더 이상 추가할 작업이 없을 때
  + `process.exit()`이나 처리되지 않은 예외로 인한 종료등 명시적인 종료에 의해 애플리케이션이 종료될 때는 **호출되지 않음** 에 유의한다.
  + `beforeExit`는 특정한 작업을 추가하려는 의도가 아니고, 단순히 `exit` 대신 사용하면 안된다.
  > 왜지

- disconnect: `Child Process`나 `Cluster Mode`로 동작시 생성된 IPC 채널이 종료되었을 때
- exit: 다음과 같은 이유로 Node의 프로세스가 종료될 때 호출됨
  + `process.exit()`가 명시적으로 호출되었을 때
  + Node의 이벤트 루프가 비어있고 더 이상 추가할 작업이 없을 때
  + 이 시점에서 이벤트 루프가 종료되는 것을 막을 방법은 없으며, 모든 `exit` 이벤트에 대한 리스너가 실행되고나면 Node의 프로세스는 종료된다.
  + `exit`의 이벤트 리스너가 모두 실행되고 나면 이벤트루프에 다른 태스크가 등록되어도 실행하지 않고 Node의 프로세스가 종료되기 때문에 리스너의 콜백함수는 **무조건** 동기적으로 실행되어야 함.
> 그럼 process.exit()를 하면 beforeExit은 호출안되고 exit만 호출?

- message: `Child Process`나 `Cluster Mode` 로 동작시 부모 프로세스로 부터 메세지를 받았을 때
- multipleResolves: `Promise` 가 다음중 하나의 일을 했을 때 마다
  + 한번이상 Resolved 되었을 때
  + 한번이상 Rejected 되었을 때
  + Resolved 된 이후 Rejected 되었을 때
  + Rejected 된 이후 Resolved 되었을 때
  + 잠재적인 버그를 찾는데 도움이 되지만, 감지된다고 무조건 버그라고 할 수는 없다. 예를들어 `Promise.race()` 는 `multipleResolves` 이벤트를 발생시킴

- rejectionHandled: 프로미스가 거절되고 에러핸들러가 Node 이벤트루프의 **한턴** 뒤에 이를 잡았을 때. 즉, 이미 거절되었던 프로미스를 조금 늦게 처리했을 때 (인것 같다).
  + `Promise` 객체는 이전에 `unhandledRejection` 때 생성되었겠지만 처리 과정에서 rejection handler 를 얻을 수 있다.
  + `Promise` 체인의 거절을 항상 처리할 수 있는 최상위 레벨의 개념은 존재하지 않는다. 기본적으로 비동기적으로 동작하기 때문에 `Promise` 의 거절은 미래시점에 발생 할 수 있으며, 이는 아마도 `unhandledRejection` 가 발생하는 이벤트 루프의 턴보다 더 늦은 시점이 될 것이다.

- uncaughtException: 처리되지 않은 자바스크립트 예외가 이벤트루프까지 전파될 때.
  + 기본적으로 Node는 이러한 예외를 스택 트레이스를 `stderr` 출력하고 이전에 설정된 `process.exitCode` 를 1로 덮어쓴 뒤, 종료시킴으로써 처리한다.
  + 위 기본동작은 'uncaughtException'에 핸들러를 등록하면 무시되어 진다.

- **'uncaughtException' 예외는 정확하게 사용되어야 한다.**
  + 'uncaughtException'는 최후의 보루 같은 조잡한 메카니즘이다. 이 이벤트는 `On Error Resume Next` 같은 처리를 위해 사용되어선 안된다. 처리되지 않은 예외는 본질적으로 애플리케이션이 정의되지 않은 상태라는 것을 의미하고, 이를 제대로 복구하지 않고 코드를 다시 시작하려 하면 예상치 못한 문제가 추가로 발생할 수 있다.
  + 무한 재귀에 빠지는걸 막기위해 이 핸들러에서 발생하는 예외는 잡히지 다시 이 핸들러로 잡히지 않는다.
  + 'uncaughtException' 의 올바른 사용은 할당받은 리소스들을 동기적으로 해제하는데 사용하는 것이다. 이외의 일반적인 연산을 이어나가는 것은 안전하지 않다.
  + 애플리케이션을 재시작 하는데에는 보다 더 신뢰성있는 방법을 사용하는 것을 추천한다.

- unhandledRejection: Promise가 거절되었고, 이벤트 루프가 한바퀴 돌았으나 이를 잡은 에러 핸들러가 존재하지 않을 때.
- warning: `process.emitWarning()` 으로 warning이 방출되었을 때.
- Signal Events
  + Node 프로세스가 시그널을 받았을 때
  + 시그널의 종류에 대해서는 [다음](http://man7.org/linux/man-pages/man7/signal.7.html)을 참고

#### 유용한 메서드 & 프로퍼티

##### process.argv
실행시 주어진 파라미터의 배열

##### process.argv0
첫번째 파라미터 - process.argv[2]를 가르킨다. process.argv[0] 은 node의 위치 process.argv[1] 은 실행할 파일 process.argv[2] 부터 파라미터이다.

##### process.channel
자식 프로세스나 클러스터 모드로 돌렸을 때 이들과 통신할 수 있는 채널에 대한 레퍼런스. 이러한 경우가 아니라면 `undefined`

##### process.env
사용자 환경에 관련된 객체를 반환한다.

이 객체는 수정될 수 있으나, 이러한 수정은 Node.js 프로세스 외부에 반영되지 않거나, (명확히 요청하지 않는 한) 다른 Worker 스레드에는 반영되지 않는다. 즉 다음 예제는 동작하지 않는다.

`$ node -e 'process.env.foo = "bar"' && echo $foo`

반면 다음 예제는 정상적으로 동작한다.

```javascript
process.env.foo = 'bar';
console.log(process.env.foo);
```

`process.env`의 새로운 프로퍼티에 값을 할당하게 되면 이 값은 문자열로 형변환된다. 이는 곧 Deprecated 될 예정이며, 앞으로의 Node 버전들에서는 문자열, 숫자, 불리언이 아닌 값은 예외를 발생하게 된다.

```javascript
process.env.test = null;
console.log(process.env.test);
// => 'null'
process.env.test = undefined;
console.log(process.env.test);
// => 'undefined'
```

`delete` 키워드를 통해 `process.env`의 프로퍼티를 삭제할 수 있다.

```javascript
process.env.TEST = 1;
delete process.env.TEST;
console.log(process.env.TEST);
// => undefined
```

Worker 인스턴스를 생성할 때 명시적으로 작성하지 않는 한, 각 Worker 스레드는 부모 스레드의 `process.env` 사본을 생성자에 갖고있다. `process.env`의 변경은 Worker 스레드 사이에서 가시적이지 않으며, 오로지 메인 스레드만이 os나 natvie add-ons에 가시적인 변경을 만들 수 있다.
