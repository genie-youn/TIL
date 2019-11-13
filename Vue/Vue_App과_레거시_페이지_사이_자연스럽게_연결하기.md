# Vue App과 레거시 페이지 사이 자연스럽게 연결하기

현재 카페는 점진적 개선중이다. 기존에 jsp로 되어있던 레거시 코드들을 Vue로 새롭게 만들어 기능들을 바꿔나가고 있다.

요즘 겪는 문제는 Vue로 구성된 SPA 에서 기존 레거시 페이지를 갔다 다시 뒤로가기 등을 통하여 돌아올 때 페이지 이동이 있기 때문에 상태고 나발이고 싹 날아가 랜더링이 새로되고, 이로인해 스크롤이 항상 맨위로 올라가는 등 안좋은 사용자 경험을 준다는 것이였는데,

팀원분이 히스토리 객체에 캐싱해두는 방법으로 멋지게 해결을 해주셔서, 이를 조금 더 추상적이고, 조금 더 다형성을 제공할 수 있도록 라이브러리화 시켰다. 이 과정에서 느꼈던 것 들을 기록해 두고자 한다.

전체 예제는 이 [저장소](https://github.com/genie-youn/vue-state-restorer) 에서 확인할 수 있다.

### 문제
아마 대부분의 컴포넌트들은 다음과 같이 구성될 것이다.

```typescript
import { Component, Prop, Vue } from 'vue-property-decorator';

@Component
export default class Travels extends Vue {
  travels = [];

  userId = 'userId';

  created() {
    this.fetchTravels();
  }

  async fetchTravels() {
    this.travels = await this.$apis.getTravels(this.userId);
  }
}
```
