# Vue프로젝트에 StoryBook 도입하기

### 들어가며
레거시들을 하나 둘 뷰로 바꿔나가면서 컴포넌트의 갯수가 점점 많아졌고. 당연히 여러 모듈에서 공통으로 사용하는 컴포넌트도 더 많아졌다. 공통으로 사용될 수 있는 컴포넌트의 효율적인 재사용과 원활한 커뮤니케이션을 위해 이를 문서화 할 수 있는 도구의 필요성을 느꼈고, 왜 많은 기업들이 아토믹 디자인이니 디자인시스템이니 하는 것들에 관심을 기울이는지 조금은 알 것 같은 느낌이다.

이 문제를 해결하기 위해 [StoryBook](https://storybook.js.org/)을 도입하기로 결정하였고 그 과정에서 겪었던 문제들을 기록해두고자 한다.

### storybook 에서 webpack이 path alias를 못찾을 때
스토리북을 처음 셋팅하고 만난 문제는 `Can't resolve '@/...'` 에러였다. vue-cli로 프로젝트를 셋팅하면 `@`를 기본적으로 프로젝트의 루트 `src`를 바라보게 웹팩 설정을 해두는데, 스토리북은 별도의 웹팩 설정을 가지므로 이를 참조할 수 없는것.

스토리북의 루트 디렉토리에 다음과 같이 웹팩 설정을 해주면 된다.

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
```

### 스토어 주입하기
두번째 문제는 스토어의 상태에 의존적인 컴포넌트에 대한 스토리를 만들때 발생하였다. 스토리북에서 보여줄 컴포넌트에 스토어는 다음과 같이 주입해주면 된다.

```JavaScript
.add('vuex + actions', () => ({
    components: { MyButton },
    template: '<my-button :handle-click="log">with vuex: {{ $store.state.count }}</my-button>',
    store: new Vuex.Store({
      state: { count: 0 },
      mutations: {
        increment(state) {
          state.count += 1;
          action('vuex state')(state);
        },
      },
    }),
    methods: {
      log() {
        this.$store.commit('increment');
      },
    },
  }))
```

> 레퍼런스틑 [다음](https://storybooks-vue.netlify.com/?path=/story/custom-method-for-rendering-vue--vuex-actions)을 참고한다.

### 스타일 로더 추가
`Module parse failed: Unexpected token (66:0)` 에러를 뱉으며 스타일을 불러오지 못할 때에는 아까 웹팩 설정의 스타일로더를 추가해주면 된다.

```JavaScript
config.module.rules.push({
    test: /\.scss$/,
    loaders: ["style-loader", "css-loader", "sass-loader"],
    include: path.resolve(__dirname, "../src/"),
});
```
