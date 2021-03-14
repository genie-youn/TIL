history back 했을 때 스크롤 위치가 정상적으로 복구가 안될 경우

라우터에 다음과 같이 추가하고 디버깅하면 원인 파악에 편리하다.

router.js
```javascript
scrollBehavior(to, from, savedPosition) {
    console.log(to, "to");
    console.log(from, "from");
    console.log(savedPosition, "savedPosition");
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(savedPosition)
      }, 300)
    })
  },
```

300ms 후에 스크롤을 복원시켰는데 정상적으로 복구가 된다면 렌더링 전에 스크롤 복원을 이미 시도했다는 얘기가 된다.
