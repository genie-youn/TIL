# ENOENT: no such file or directory, rename~/.staging/ 에러가 발생할 경우

`node_modules` `package-lock.json` 을 제거하고 다시 npm install 을 해주면 된다.

```shell
$ npm cache clean --force
$ rm -rf ./node_modules
$ rm package-lock.json && npm i
```
