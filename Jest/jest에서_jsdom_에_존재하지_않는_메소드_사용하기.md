# jest에서 jsdom에 존재하지 않는 메소드를 사용하고 싶다면?

`jest` 를 통해 단위테스트를 작성하다보면 DOM API를 사용해야 할 일이 생긴다.
예를 들어 테스트 코드 내에서 `window.matchMedia()`를 사용하면 다음과 같은 예외를 만날 수 있다. `TypeError: window.matchMedia is not a function``

그 이유는 `jest`가 dom 구현체로 사용하는 `jsdom` 에는 `window.matchMedia` 가 존재하지 않기 때문인데, 다음과 같이 mocking 하여 사용하면 된다.

```javascript
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(), // deprecated
    removeListener: jest.fn(), // deprecated
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

> 자세한 내용은 [레퍼런스](https://jestjs.io/docs/en/manual-mocks#mocking-methods-which-are-not-implemented-in-jsdom)를 참고.
