### ReactiveKafkaTemplate

카프카 스터디를 하다가 spring-kafka가 언제쯤 reactive-streams 구현체를 지원할지 궁금해져서 찾아봤다.
v2.3 부터 지원할 예정이고 현재 2.3.0 M1이 깨져있는 상황이였다. 아마 조만간 볼 수 있을듯?

스프링 프로젝트답게 코어 라이브러리로는 reactor 를 사용했다. 정확히는 [reactor-kafka](https://github.com/reactor/reactor-kafka) 를 사용한다.
2016년부터 해당 논의가 시작되었는데, 과정이 흥미로웠다. 

Pivotal의 스프링커미터가 혜성처럼 등장해 이슈업을 하고 spring-kafka의 maintainer들과 협력하며 이슈를 클리어해 가는 과정이
오픈소스 프로젝트의 매력을 느끼게 했다. 한번쯤 따라가며 읽어봐도 좋은 쓰레드

논의스레드 https://github.com/spring-projects/spring-kafka/issues/43

관련 PR https://github.com/spring-projects/spring-kafka/pull/559

2.3.0 M1 릴리즈 노트 https://github.com/spring-projects/spring-kafka/releases/tag/v2.3.0.M1
