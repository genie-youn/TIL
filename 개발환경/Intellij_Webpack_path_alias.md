# Intellij 에서 webpack path alias 적용하기

#### Webpack path alias

Webpack 은 다른 모듈을 불러오는걸 좀 더 간편하게 하기 위해 다음과 같은 설정을 제공한다. 예를들면

webpack.config.js
```javascript
module.exports = {
  //...
  resolve: {
    alias: {
      @: path.resolve(__dirname, 'src/'),
      @unit: path.resolve(__dirname, 'test/unit')
      @functions: path.resolve(__dirname, 'src/commons/utils/functions')
    }
  }
};
```

불필요하게 `src/components` 이런 경로가 반복되서 나타나지 않으니 참 간편하긴 한데, 문제는 intellij 에서 이렇게 불러올 경우 해당 모듈을 찾지도 못하고, 그로 인해 code assistant 도 동작하지 않는다..

해결책은 간단하다. intellij 에 해당 웹팩 설정파일 경로를 잡아주면 된다.

`Preference` > `Languages&Frameworks` > `Javascript` > `Webpack` 에 `webpack.config.js` 파일의 경로를 넣어주면 잘 찾아준다.

![image](https://user-images.githubusercontent.com/16642635/54926134-0cf87480-4f53-11e9-9dfa-266c06b50d03.png)
