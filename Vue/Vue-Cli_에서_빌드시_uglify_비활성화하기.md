# Vue-Cli에서 빌드시 uglify 비활성화 하기

uglify는 코드가 클라이언트에서 실행되는 프론트엔드 특성상 꼭 필요한 과정이지만, 디버깅할 때 큰 걸림돌이 된다.
vue-cli를 사용중이라면 다음과 같이 설정하여 빌드 시 비활성화 시킬 수 있다.

vue.config.js
```javascript
module.exports = {
  chainWebpack: config => config.optimization.minimize(false)
}
```
