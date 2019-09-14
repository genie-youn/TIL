# Axios에서 Global inherited Interceptor 구현하기

## Interceptor
Axios에서 요청을 보냈을 때나, 응답을 받았을 때 그 값을 가로채 무언가 액션을 취하려면 `interceptor` 를 사용하면 된다.
이 `interceptor`는 두 가지 방법으로 사용 가능하다.

글로벌 스코프에 설정해서 쓰거나
```javascript
import axios from "axios";
// Add a request interceptor
axios.interceptors.request.use(function (config) {
    // Do something before request is sent
    return config;
  }, function (error) {
    // Do something with request error
    return Promise.reject(error);
  });

// Add a response interceptor
axios.interceptors.response.use(function (response) {
    // Do something with response data
    return response;
  }, function (error) {
    // Do something with response error
    return Promise.reject(error);
  });
```

아니면 인스턴스에 선언해서 사용하거나
```javascript
const instance = axios.create();
instance.interceptors.request.use(function () {/*...*/});
```

## 문제
문제는 HTTPRequest를 보낼 서버가 여러개고 이 응답들을 내부적인 프로토콜을 따르고 있어 동일한 인터셉터로 처리할 수 있을 때이다. `axios`의 기본 설정 객체인 `defaults`는 글로벌 스코프에 선언하면 인스턴스 생성시 상속받게 된다.

```javascript
axios.defaults.baseURL = "http://a.com";
const instance1 = axios.create();
const instance2 = axios.create();
console.log(instance1.defaults.baseURL);  // 'http://a.com'
console.log(instance2.defaults.baseURL);  // 'http://a.com'
instance1.defaults.baseURL = "http://b.com";
console.log(instance1.defaults.baseURL);  // 'http://b.com'
console.log(instance2.defaults.baseURL);  // 'http://a.com'
```

하지만 인터셉터의 경우 상황이 조금 다르다. 글로벌 스코프의 인터셉터가 인스턴스로 상속이 되지 않기 때문에 Axios 인스턴스를 생성할 때마다 인터셉터를 생성해주어야 한다.

```javascript
axios.interceptors.response.use(response => response, error => Promise.reject);
const instance = axios.create();

console.log(axios.interceptors.response.handlers.length);  // 1
console.log(instance.interceptors.response.handlers.length); // 0
```

글로벌 스코프의 인터셉터를 상속받을 수 있는 옵션을 추가해 달라는 [이슈](https://github.com/axios/axios/issues/993)가 올라와 있지만 진행될 기미는 보이지 않는다. 그래서 우선 기존의 `create`를 확장하여 사용하기로 하였다.

AxiosDecorator.js
```javascript
import axios from "axios";

axios.defaults.baseURL = "https://a.com";
axios.defaults.timeout = 3000;
axios.defaults.withCredentials = true;

axios.interceptors.response.use(response => response, Promise.reject);

const create = config => {
  const instance = axios.create(config);
  instance.interceptors.request.handlers = [...axios.interceptors.request.handlers];
  instance.interceptors.response.handlers = [...axios.interceptors.response.handlers];
  return instance;
};

const axiosDecorator = {
  create
};

export default axiosDecorator;
```

근데 이러면 global scope에서 상속받은 핸들러는 어떻게 지워야 좋지? 몇개 상속받았는지 알아내서 지워줘야 하나.
인터셉터를 해제하는 `eject` 함수는 인덱스를 인자로 받아서 해당하는 인덱스의 핸들러를 삭제한다. 그럼 핸들러 등록했을 때의 인덱스를 계속 들고있다가 필요해지면 삭제를 해야하는건가?

```javascript
/**
	 * Remove an interceptor from the stack
	 *
	 * @param {Number} id The ID that was returned by `use`
	 */
	InterceptorManager.prototype.eject = function eject(id) {
	  if (this.handlers[id]) {
	    this.handlers[id] = null;
	  }
	};
```

- 글로벌 인터셉터는 런타임에 상속받아야 하는가 아니면 생성될때 복사되어야 하는가?
- 런타임에 상속받게 된다면 인터셉터들의 순서는 어떻게 되어야 하는가?
