스프링 부트의 버전을 2.0.x 에서 2.1.x 로 올렸더니
`org.elasticsearch.search.SearchHit#getSource` 메소드를 찾을수 없다는 에러를 뱉기 시작했다.

`SearchHit` 클래스의 깃 히스토리를 쭉 보다가 [PR #23042](https://github.com/elastic/elasticsearch/pull/23042) 에서 해당 메소드 시그니처가 삭제된걸 찾을 수 있었고 해당 피쳐는 `v6.0.0-alpha1` 부터 적용되었다. (라벨은 `v5.4.0` 도 붙어있던데 뭐지)

 부트의 버전이 올라가면서 `spring-boot-starter-data-elasticsearch` 의 버전이 `2.0.2` 에서 `2.1.1` 로 
이에 따라서 `spring-data-elasticsearch` 의 버전이 `3.0.7` 에서  `3.1.3` 으로
`org.elasticsearch.elasticsearch` 의 버전이 `5.5.0` 에서 해당 피쳐가 반영된 `6.4.3` 으로 올라가면서 생긴 문제인것 같다.



기존에 `SearchHit#getSource` 를 사용하는 부분을 `SearchHit#getSourceAsMap` 으로 바꿔주면 기존과 똑같이 동작한다. 이걸 왜 이렇게 바꿧나 봤더니 기존에 `source` 라는 필드는 `byte[]` (.. 였고) `getSource()` 와 `source()` 가 동시해 존재했으며 심지어 `getSource()` 는 `Map<String,Object>` 를 반환하고 있었다..  `getSourceAsMap()` 도 이미 존재하고 있었고. 이 인터페이스를 구현하고 있는건 `InternalSearchHit.java` 하나의 클래스였어서 인터페이스를 제거하고 합쳐서 클래스로 만들어버린것. 그 과정에서 메소드 시그니처를 다듬은것 같다.



구현할때 자세히 보지 않고 코드 자동완성의 도움의 받아 `Map<String,Object> getSource()` 라는 시그니처만 보고 가져다 쓰지 말자 😂



\#PR https://github.com/elastic/elasticsearch/pull/23042
\#Commit https://github.com/elastic/elasticsearch/commit/ecb01c15b9a6645f22f153eb099a377e70e398c8



\#태그 `elasticsearch` `elasticsearch 버전` `SearchHit#getSource` 