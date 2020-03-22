# Vue-Cli의 index.html 에서 javascript 변수를 사용하고 싶다면?

개발을 하다보면 사용자에게 서빙하는 `index.html` 에도 자바스크립트 변수를 사용해야 하는 상황이 생기기 마련이다.

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

`20200322` 이 값을 빌드하는 날짜에 맞춰서 변경해주고 싶을텐데. 빌드할대 마다 값을 바꿔줄 수 도 없는 노릇이다.

이때 vue-cli 를 통해 개발환경을 구성했다면, `favicon`의 URL을 지정해 주었듯, [lodash 의 template](https://lodash.com/docs/4.17.15#template) 형식을 통해 값을 index.html에 끼워넣을 수 있다.

```html
<script src="https://test.library.url?v=<%= new Date().toISOString().slice(0,10).replace(/-/g,'') %>"></script> <!-- 이부분 -->
```

> 자세한 내용은 [레퍼런스](https://cli.vuejs.org/guide/html-and-static-assets.html)를 참고
