# 12corp-search-doc

# 검색 서버 문서

## 검색서버 관련 주요 구성요소(사용기술)

* nodejs + express :
* elasticsearch: fulltext search engin으로 실질적인 검색 DB 역할을 함.
* mecab: 자연어 검색을 위한 한글 자소 분석 모듈. (이 모듈 및 사전관련해서는 루시가 전문가임.)
* mongodb: 색인된 데이터를 가져오기 위한 저장소역할.
* 검색 로직 구현 모듈: pasta>pickle

---
## 검색 관련 툴

* marval : elasticsearch 쿼리를 직접 날려볼 수 있는 웹콘솔(Oracle의 SQL*Plus or MySQL의 Workbench)
 - http://api.pasta.12corp.com:19200/_plugin/marvel/sense/
 - http://dev.pasta.12corp.com:19200/_plugin/marvel/sense/

 - 주요 쿼리:
   ```json
   [모든 인덱스 보기]
   GET _cat/indices?v

   [인덱스 필터링 해서 보기]
   GET _cat/indices/yap_place2016050*?v

   GET _cat/indices/yap_place_ac2016050*?v

   [모든 alias 보기]
   GET /_aliases

   [alias 필터링 해서 보기]
   GET /_alias/yap_place*

   [alias 추가 및 삭제]
   GET /_aliases
   {
       "actions" : [
           { "add" : { "index" : "yap_place201603031512", "alias" : "yap_place" } }
       ]
   }'

   GET /_aliases
   {
       "actions" : [
           { "remove" : { "index" : "yap_place201603010212", "alias" : "yap_place" } }
       ]
   }'

   [특정 인덱스 삭제]
   DELETE /yap_place_ac2016041*
   DELETE /yap_place2016041*

   [검색 쿼리]
    GET /yap_place/PlaceIndex/_search
    {
      "filter": {
        "and": [
          {
            "term": {
              "areas.id": 600301
            }
          }
        ]
      },
      "from": 0,
      "size": 10,
      "sort": [
        {
          "score.total": {
            "order": "desc"
          }
        },
        {
          "score.ps": {
            "order": "desc"
          }
        },
        {
          "score.checkinFreq": {
            "order": "desc"
          }
        }
      ]
    }
   ```

---
### 검색 테스트 클라이언트
 #####  12corp 검색을 직접 수행해보기 위해 개발 된 편의 툴.

 * http://api.pasta.12corp.com/#!/pickle
 * http://dev.pasta.12corp.com/#!/pickle

    - napoli>pickle>search (검색 서버 모듈)
    - napoli>pasta>search (검색 테스트 클라이언트 모듈)
    - queryType(xxx)별 pickle/search/qm/xxx-query-maker.js 할당.
    - resultType (result-data-maker.js)
    - xxx-controller.js

---

### 로그
 * `/home/apps/logs`: (cdl)기본 pasta 로그 디렉토리
 * `/home/apps/napoli/_shared/log`: (cdll) 검색 서버관련 특화된 로그 분석용 디렉토리
 * slack napoli_log: 주시해야 할 서버 오류
 * slack napoli_fatal_error: 필히 수정되야 할 서버 오류
 * `napoli>_shared/log/logger.js` 모듈에서 로그 레벨 및 slack forwarding관리되고 있음.

---

### 자동완성

    - napoli>pickle>search (검색 서버 모듈)

---
### 메모리로드 사전

---
