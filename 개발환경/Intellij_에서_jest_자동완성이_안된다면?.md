# Intellij에서 Jest가 자동완성이 안된다면?

## 들어가며
얼마 전, 테스트코드를 작성하다가 "왜 Jest가 자동완성이 안되지?" 하는 생각이 들었고 플러그인을 받아줘야 하나 하고 마켓플레이스에 찾아봤지만 Jest관련 플러그인은 존재하지 않았다.. 없을리가 없는데 하고 검색하던중 Intellij에 자바스크립트 라이브러리들의 타입정보를 넣어주는 기능이 있어 정리해두려한다.

## 방법
Preferences > Languages & Frameworks > JavaScript > Libraries 로 이동한다.
![](https://user-images.githubusercontent.com/16642635/82499361-83c7d800-9b2c-11ea-9192-25ed7e11b2f8.png)

오른쪽 다운로드를 누르면 타입을 설치할 수 있는 라이브러리들이 주르륵 나온다. 여기서 jest를 검색.
![](https://user-images.githubusercontent.com/16642635/82499365-875b5f00-9b2c-11ea-936b-2e30c24f7fe8.png)

Download and Install을 누르면 다음과 같이 추가되는걸 볼 수 있다.
![](https://user-images.githubusercontent.com/16642635/82499373-89252280-9b2c-11ea-9cbf-61ee676808cf.png)

이제 자동완성 가능!
![](https://user-images.githubusercontent.com/16642635/82500008-9e4e8100-9b2d-11ea-8bf2-29368011bce0.png)

## 마치며
사용하고 있는 라이브러리가 뭔가 자동완성이 잘 안된다 싶으면 `Preferences > Languages & Frameworks > JavaScript > Libraries` 에서 해당 라이브러리가 있는지 찾아서 설치해보자. 없다면 따로 제공하고 있는 플러그인이 있는지.. 플러그인도 없으면 스스로 만들..

> 참고 https://blog.jetbrains.com/webstorm/2018/10/testing-with-jest-in-webstorm/
