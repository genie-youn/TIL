# Vue-Cli의 index.html 에서 javascript 변수를 사용하고 싶다면?

## 들어가며
개발을 하다보면 사용자에게 서빙하는 `index.html` 에도 자바스크립트 변수를 사용해야 하는 상황이 생기기 마련이다.

이 경우 `vue-cli`를 통하여 프로젝트를 구성했다면, `index.html`에 [lodash 의 template](https://lodash.com/docs/4.17.15#template) 형식을 통해 값을 index.html에 끼워넣을 수 있다.

예제를 통해 확인해 보자.

## 예제
예를들면 CDN 으로 제공하는 사내 다른 라이브러리의 캐시 퍼징을 위해 suffix로 빌드시점의 날짜를 받게 되어있다고 가정해보자.

아마 다음과 같은 모습일 것이다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
    <script src="https://test.library.url?v=20200322"></script> <!-- 이부분 -->
  </body>
</html>
```

`20200322` 이 값을 빌드하는 날짜에 맞춰서 변경해주고 싶을텐데. 빌드할때 마다 값을 바꿔줄 수 도 없는 노릇이다.

이때 vue-cli 를 통해 개발환경을 구성했다면, `favicon`의 URL을 지정해 주었듯, 자바스크립트 변수를 `<%= =%>`로 감싸서 사용할 수 있다.


```html
<script src="https://test.library.url?v=<%= new Date().toISOString().slice(0,10).replace(/-/g,'') %>"></script>
```

lodash의 template는 이 외에도 몇가지 보간법을 사용할 수 있다.

```javascript
// <%- -%>는 감싸진 값을 HTML escape 시킨다.
var compiled = _.template('<b><%- value %></b>');
compiled({ 'value': '<script>' });
// => '<b>&lt;script&gt;</b>'

// <% %> 는 내부의 자바스크립트 표현식을 평가한다.
var compiled = _.template('<% _.forEach(users, function(user) { %><li><%- user %></li><% }); %>');
compiled({ 'users': ['fred', 'barney'] });
// => '<li>fred</li><li>barney</li>'

// ES6의 template literal 형식도 사용 가능하다.
var compiled = _.template('hello ${ user }!');
compiled({ 'user': 'pebbles' });
// => 'hello pebbles!'

// 문자열 내에서 escape 하려면 백슬래쉬를 사용하면 된다.
var compiled = _.template('<%= "\\<%- value %\\>" %>');
compiled({ 'value': 'ignored' });
// => '<%- value %>'
```


> 자세한 내용은 [레퍼런스](https://cli.vuejs.org/guide/html-and-static-assets.html)를 참고
