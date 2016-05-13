# 12corp-search-doc

# 검색 서버 문서

---
### 검색서버 관련 주요 구성요소(사용기술)

* nodejs + express :
* elasticsearch: fulltext search engin으로 실질적인 검색 DB 역할을 함.
* mecab: 자연어 검색을 위한 한글 자소 분석 모듈. (이 모듈 및 사전관련해서는 루시가 전문가임.)
* mongodb: 색인된 데이터를 가져오기 위한 저장소역할.
* 검색 로직 구현 모듈: pasta>pickle

* 검색의 일련의 처리과정
 '검색어' 서버 전송 -> 검색어에 따래 해당 query-manager를 채택 -> 검색어 분석(mecab등) 및 키워드 추출 ->
 elasticsearch query 생성 -> elasticsearch query 실행 -> 결과를 data-result-maker로 필터링해서 반환.

---
### 검색 서버 접속

* 검색 개발 서버 접속(node + mongodb + elasticsearch): pasta admin / 검색 개발 서버
    - ssh -i .ssh/12corp.pem ec2-user@dev.pasta.12corp.com

* 검색 리얼 서버 접속(node + elasticsearch): 검색 담당
    - ssh -i .ssh/12corp.pem ec2-user@api.pasta.12corp.com

* 파스타 리얼 서버 접속(node + mongodb): pasta admin 담당
    - ssh -i .ssh/12corp.pem ec2-user@pasta.12corp.com

```sh
    aws 이전 password 방식으로 접속했을 때:
    #/usr/local/bin/sshpass -p 'dufentl!' ssh admin@dev.pasta.12corp.com
    #/usr/local/bin/sshpass -p '12Ghkdlxld!@#' ssh admin@api.pasta.12corp.com
    #/usr/local/bin/sshpass -p '12Ghkdlxld!@#' ssh admin@pasta.12corp.com
```
---
### 검색 관련 툴

* marval : elasticsearch 쿼리를 직접 날려볼 수 있는 웹콘솔(Oracle의 SQL*Plus or MySQL의 Workbench)
 - http://api.pasta.12corp.com:19200/_plugin/marvel/sense/
 - http://dev.pasta.12corp.com:19200/_plugin/marvel/sense/

 - index, type의 개념.

 - 주요 쿼리:
   ```json
   "* 모든 인덱스 보기]"
   GET _cat/indices?v

   "* 인덱스 필터링 해서 보기"
   GET _cat/indices/yap_place2016050*?v

   GET _cat/indices/yap_place_ac2016050*?v

   "* 해당 인덱스의 타입 및 각 컬럼의 type보기"
   GET /yap_place/_mapping
   GET /yap_place_ac/_mapping

   "* 모든 alias 보기"
   GET /_aliases

   "* alias 필터링 해서 보기"
   GET /_alias/yap_place*

   "* alias 추가 및 삭제"
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

   "* 특정 인덱스 삭제"
   DELETE /yap_place_ac2016041*
   DELETE /yap_place2016041*

   "* 검색 쿼리"
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
### 검색 서버 태스크
 * 검색 서버 시작, 재시작, 종료
    /home/ec2-user/pizza-xxxx.sh

 * napoli home 디렉토리 `/home/apps/napoli` (cdn)
    - 소스코드 갱신이 필요하면 여기서 >git pull을 하고 바로 >restart_all로 전체 서버의 재시작을 할 수 있음.

 * 검색 색인 배치 디렉토리 `/home/apps/napoli/bacon/indexing` (cdb)
    - z-index-xxx.sh: 검색 색인 배치 처리 실행 (mySQL -> mongoDB -> elasticsearch)
    - reload-memory-dict.sh: 각 서버에 키워드/테그 사전 최신으로 갱신.(검색 서버 재시작 없이 반영됨.)

---

### 검색 테스트 클라이언트
 #####  12corp 검색을 직접 수행해보기 위해 개발 된 편의 툴.

 * http://api.pasta.12corp.com/#!/pickle
 * http://dev.pasta.12corp.com/#!/pickle

    - `napoli/pickle/search` (검색 서버 모듈)
    - `napoli/pasta/search` (검색 테스트 클라이언트 모듈)
    - queryType(xxx)별 `pickle/search/qm/xxx-query-maker.js` 할당.
    - resultType (`pickle/search/esult-data-maker.js`)


 * 주요 검색 서버 controllers
    - `pickle/napoli-pickle-controller.js` : 검색 및 검색과 관련된 주요 처리 담당.
    - `pickle/auto-complete-controller.js` : 자동완성 데이터 요청 처리 담당.
    - `pickle/relative-store-controller.js` : 연관검색어 처리 담당.
    - `pickle/filter-data-controller.js` : 검색어 필터 항목 제공(현재는 테스트 클라이언트에서만 사용함)
---

### 로그
 * `/home/apps/logs`: (cdl)기본 pasta 로그 디렉토리
 * `/home/apps/napoli/_shared/log`: (cdll) 검색 서버관련 특화된 로그 분석용 디렉토리
 * slack napoli_log: 주시해야 할 서버 오류
 * slack napoli_fatal_error: 필히 수정되야 할 서버 오류
 * `napoli/_shared/log/logger.js` 모듈에서 로그 레벨 및 slack forwarding관리되고 있음.
 * 검색 테스트 클라이언트에서 log level을 서버 재시작 없이 실시간으로 변경 가능

 * hookURL 만들기
    - https://api.slack.com/incoming-webhooks 에서 'incoming webhook integration' 부분을 클릭해서 새로 만들 수 있음.

 * 12corp의 hook리스트보기
    - https://12corp.slack.com/apps/manage/A0F7XDUAZ-incoming-webhooks
---
### 사전
 * 사전의 종류
    - mecab 사전 (관리자: Lucy)
    - 키워드/태그 사전: pasta mongoDB에서 관리됨
    - analized 키워드/태그 사전: 검색어 분석처리하여 메모리에 로딩해두고 사용함.


 * 메모리로드 사전(analized 키워드/태그 사전)
    - 기획의 복잡한 검색 알고리즘을 단순화 하고 검색어 분석 처리의 속도 향상 위해 키워드/테그 사전을 분석하여 메모리에 로딩해서 관리하는 사전.
    - `napoli/bacon/indexing/reload-memory-dict.sh` 를 실행해서 생성하고 현재 실행중인 모든 서버 메모리에 갱신해줌.
    - `napoli/data/dictData.json` 와 mongoDB의 napoli-spoon.DictData 에 통 json 파일로 저장됨.
    - 백업되어 있는 파일 사전데이터 있으면 파일 사전데이터를 읽어들이고 없으면 DB에서 읽어들임.
    - dictData.json을 읽어들이면서 각 키워드별 우선순위에 따른 분석을 거쳐 분석된 사전을 메모리에 만들어 둠.
    - 예) '신사동에서 가장 맛있는 파스타집' -> '신사동' '파스타'로 분리 -. '신사동'을 분석된 메모리로드 사전에서 검색하면
      동의어, 지역 레벨, 매장 갯수를 고려해서 가장 적합한 것을 가져오게 됨.

---

### 자동완성
 * 자동완성은 2단어까지만 자동완성을 지원함.
 * 지역 키워드와 비지역 키워드를 조합해서 추천함. (신사동 ㅍ-> 신사동 파스타, 파스타 ㅅ-> 파스타 신사동)
 * 초성검색 지원
 * 자동완성 검색용 데이터 생성 `/home/apps/napoli/bacon/indexing/make-auto-comp-data.js`
    - node make-auto-comp-data.js -c xxx (loc / dev / prd)

---
### node 버전 업그레이드(prd서버)
 * napoli git branches
    - master: trunk 에 해당.
    - dev: DEV 에 해당
    - nodeUpgrade: local 및 dev 서버에 올려서 테스트 할 용.


 * 현재 node 5.x version upgarde를 dev 서버까지만 함.
 * production 서버 node를 최신으로 upgrade 하려면 package.json에서 elasticsearch와 ffi로 변경하고 해야 함.
    ```json
        {
        ...
        "elasticsearch": "^10.0",
        "ffi": "^2.0",
        ...
        }
    ```

--
