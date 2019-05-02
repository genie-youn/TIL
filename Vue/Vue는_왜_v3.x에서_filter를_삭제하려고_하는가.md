# Vue는 왜 v3.x에서 filter를 삭제하려고 하는가.

지나가다 `filter`에 값이 변경될 때만 다시 랜더링하게 해달라는 이슈를 보았고, 그 댓글에 뷰의 메인테이너인 `Evan You`가 `filter` 를 3.0에서 삭제하고 싶었지만, 2.0에서 삭제되었다가 커뮤니티의 요청으로 가장 일반적인 문법만이 다시 추가되었다는 이야기를 남긴걸 보고 2.0 설계 쓰레드에서 왜 `filter` 가 삭제될 뻔 했는지를 따라가 보았다.

> 쓰레드 원글은 https://github.com/vuejs/vuex/issues/236 여기를 참고하면 된다.
> 추가로 본래 이슈는 https://github.com/vuejs/rfcs/pull/6 여기를 참고하면 된다.
