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
