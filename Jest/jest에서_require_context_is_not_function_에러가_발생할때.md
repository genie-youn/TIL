# Jest에서 Require.context is not function 이라는 에러가 발생할 경우

# 해결
[babel-plugin-transform-require-context](https://www.npmjs.com/package/babel-plugin-transform-require-context) 플러그인을 설치한 후에
```shell
npm install -dev babel-plugin-transform-require-context
```

`.babelrc` 혹은 `babel.config.js` 에 플러그인을 추가해주면 된다.

```json
{
  "env": {
    "test": {
      "plugins": ["transform-require-context"]
    }
  }
}
```
