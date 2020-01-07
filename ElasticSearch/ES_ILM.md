# ElasticSearch Index Life-cycle management

## 들어가며
액세스 로그 수집을 위한 ES 클러스터를 셋팅하면서 당연히 해줄것이라 예상했던 인덱스의 보관기간을 관리하는 기능이 없다는걸 알게되고, 적잖이 충격을 받아 그 내용을 정리해두려 한다.

## ILM (Life-Cycle management)
ES는 ILM이란 기능을 통해 인덱스의 보관 기간을 관리할 수 있다. 문제는 이 기능이 `X-Pack` 포함기능이라, 오픈소스버전의 ES에선 사용할 수 없다는것. `X-Pack` 을 사용하고 있다면 다음과 같이 간단하게 설정할 수 있다.

```
PUT _ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "25GB"
          }
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

오픈소스버전을 사용중이여서 위 기능을 사용하지 못한다면.. 어쩔수 있겠는가. 여러 서드파티들이 있는것 같았지만 복잡시러워서 serverless 플랫폼을 통해서 삭제 배치를 구현해두고 일배치를 돌게 하였다.

> https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html

## Authenticate
인덱스를 삭제한 후 상태를 조회해 디스크 여유공간은 어떤지 등의 정보를 사내 메신저로 리포팅하도록 구현하였는데 각각 다음 API를 사용하면 된다.

#### 삭제
```
DELETE /twitter
```

> https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-delete-index.html

#### 상태조회
```
GET /_cluster/stats?human&pretty
```

> https://www.elastic.co/guide/en/elasticsearch/reference/current/cluster-stats.html

API를 호출하기 위해서는 인증정보가 필요한데, 간단하게 `id:password`를 base64 인코딩하여 요청의 `Authorization` 헤더에 넣어주면 된다.
자바스크립트를 사용하고 있다면,

```javascript
const authKey = btoa('myESID:myPassword');
fetch(url, {
    method: 'POST',
    headers: {
      'Authorization': `Basic ${btoa('myID:myPassword')}`
    },
  });
```

정도가 되겠다.
