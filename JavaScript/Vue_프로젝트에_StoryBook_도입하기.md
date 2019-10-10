# Vue프로젝트에 StoryBook 도입하기

### 들어가며
레거시들을 하나 둘 뷰로 바꿔나가면서 컴포넌트의 갯수가 점점 많아졌고. 당연히 여러 모듈에서 공통으로 사용하는 컴포넌트도 더 많아졌다. 공통으로 사용될 수 있는 컴포넌트의 효율적인 재사용과 원활한 커뮤니케이션을 위해 이를 문서화 할 수 있는 도구의 필요성을 느꼈고, 왜 많은 기업들이 아토믹 디자인이니 디자인시스템이니 하는 것들에 관심을 기울이는지 조금은 알 것 같은 느낌이다.

이 문제를 해결하기 위해 [StoryBook](https://storybook.js.org/)을 도입하기로 결정하였고 그 과정에서 겪었던 문제들을 기록해두고자 한다.


# storybook 에서 webpack이 path alias를 못찾을 때
 Can't resolve '@/...'

```JavaScript
const path = require("path");

module.exports = ({config}) => {
    config.module.rules.push({
        test: /\.vue$/,
        loader: "storybook-addon-vue-info/loader",
        enforce: "post"
    });

    config.resolve = {
      ...config.resolve,
    alias: {
        ...config.resolve.alias,
      "@": path.resolve(__dirname, "../src/"),
    },
  };

    return config;
};

스토어는 일케 주면 된다.
https://github.com/storybookjs/storybook/issues/4259
https://storybooks-vue.netlify.com/?path=/story/custom-method-for-rendering-vue--vuex-actions

Module parse failed: Unexpected token (66:0)
File was processed with these loaders:
라고 하며 스타일 못읽을 땐 이렇게

config.module.rules.push({
    test: /\.scss$/,
    loaders: ["style-loader", "css-loader", "sass-loader"],
    include: path.resolve(__dirname, "../src/"),
});
