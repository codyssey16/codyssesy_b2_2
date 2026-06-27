# ▶️ 각종 실행 로그

## 1. 노션 DB에 뉴스 저장 성공

실행 완료:

<img src="../images/success.jpg" width="800"/>

<br>

노션 DB 저장 테이블:

<img src="../images/success_notion_table.jpg" width="800"/>

<br>

노션 DB 저장 결과:

<img src="../images/success_notion_page.jpg" width="500"/>

---

## 2. RSS 에러 알림

RSS 에러 상황 테스트: RSS 모듈의 URL란에 없는 주소를 넣음. 예) `https://www.techcrunch.com/fe`

실행 완료:

<img src="../images/error_rss.jpg" width="800"/>

<br>

이메일 전송:

<img src="../images/error_rss_email.jpg" width="500"/>

---

## 3. AI 관련 기사 없을 시 구글 시트에 기록

실행 완료:

<img src="../images/issue_no_AI_news.jpg" width="800"/>

<br>

구글 시트 기록:

<img src="../images/issue_no_AI_news_sheets.jpg" width="500"/>

---

## 4. Notion 에러 이메일 알림 후 재시도

노션에 에러가 났을 때 error handler에서 Router를 넣어서 Path A를 치명적인 에러에 이메일을 보내고 재시도, 그리고 Path B를 일반적인 에러에 구글 시트의 issue_log에 넣고 재시도를 하게 해도 되지만, 현 과제를 위해서 간단하게 모든 에러에 이메일을 보내고 재시도를 하게 함.

노션 에러 상황 테스트: Data Source ID를 없는 ID로 넣고 실행.

### <첫 번째 Notion>

실행 완료: 

<img src="../images/error_notion1.jpg" width="800"/>

<br>

이메일 전송:

<img src="../images/error_notion1_email.jpg" width="500"/>

### <두 번째 Notion>

실행 완료:

<img src="../images/error_notion2.jpg" width="800"/>

<br>

이메일 전송:

<img src="../images/error_notion2_email.jpg" width="500"/>

---

## 5. OpenAI 에러 이메일 알림 후 재시도

오픈AI에 에러가 났을 때 error handler에서 Router를 넣어서 Path A를 치명적인 에러에 이메일을 보내고 재시도, 그리고 Path B를 일반적인 에러에 구글 시트의 issue_log에 넣고 재시도를 하게 해도 되지만 현 과제를 위해서 간단하게 모든 에러에 이메일을 보내고 재시도를 하게 함.

오픈AI 에러 상황 테스트: `Model`필드에서 `Map`을 선택하고 없는 GPT모델을 넣음. 예) `gpt-fake-model-12345` 

### <첫 번째 OpenAI>

실행 완료:

<img src="../images/error_OpenAI1.jpg" width="800"/>

<br>

이메일 전송:

<img src="../images/error_OpenAI1_email.jpg" width="500"/>

### <두 번째 OpenAI>

실행 완료:

<img src="../images/error_OpenAI2.jpg" width="800"/>

<br>

이메일 전송:

<img src="../images/error_OpenAI2_email.jpg" width="500"/>

---

## 6. HTTP 에러 구글 시트에 이슈 기록

`HTTP` 에서 에러가 났을 때는 즉각적인 조치가 필요하지 않기 때문에 알림을 보내지 않고 추적 및 감사를 하기 위해 구글 시트에 이슈에 대한 행 삽입. 그리고 `HTTP` -> `Text Parser` 중간에 필터를 사용하여 아래 이미지와 같이 `Status Code` 가 200~299일 때만 다음 모듈(`Text Parser`)로 가게 함.

<figure style="text-align: center;">
  <img src="../images/filter_http_input.jpg" alt="HTTP Filter Input" width="300"/>
  <figcaption>HTTP 모듈 바로 뒤의 필터 조건. Status Code가 200~299여야 한다.</figcaption>
</figure>

HTTP 에러 상황 테스트: URL을 첫 번째 OpenAI에서 나온 출력이 아닌 `https://doesntexist.com`으로 넣고 실행.

실행 완료:

<img src="../images/error_http.jpg" width="800"/>

<br>

구글 시트 기록:

<img src="../images/error_http_sheets.jpg" width="500"/>

---

## 7. Parse JSON가 JSON형식의 string을 받지 않은 경우 구글 시트에 이슈 기록

`Parse JSON` 에서 에러가 났을 때도 즉각적인 조치가 필요하지 않기 때문에 알림을 보내지 않고 추적 및 감사를 하기 위해 구글 시트에 이슈에 대한 행 삽입.

Parse JSON 에러 상황 테스트: `JSON string`필드에 OpenAI가 출력한 JSON 형식의 답이 아닌 `not-json-you-want`을 넣고 실행.

실행 완료:

<img src="../images/error_parseJSON.jpg" width="800"/>

<br>

구글 시트 기록:

<img src="../images/error_parseJSON_sheets.jpg" width="500"/>

---

## 8. Parse JSON에서 모든 데이터를 받지 않았을 때 구글 시트에 이슈 기록

Parse JSON 상황 테스트: `JSON string`필드에 OpenAI가 출력한 JSON 형식의 답에서 `Summary`를 빼고 실행.

`JSON string` 필드 입력 예:
```
{"Title": "Patronus AI, AI 에이전트 스트레스 테스트용 ‘디지털 월드’ 구축 위해 5,000만 달러 투자 유치","URL": "https://techcrunch.com/2026/06/25/patronus-ai-lands-50m-to-build-digital-worlds-that-stress-test-ai-agents/","Date": "2026-06-25 20:19:25","Emotion": "긍정","Guid": "https://techcrunch.com/?p=3136499"}
```

실행 완료:

<img src="../images/issue_parseJSON.jpg" width="800"/>

<br>

구글 시트 기록:

<img src="../images/issue_parseJSON_sheets.jpg" width="500"/>

---

## 9. 기사 중복 체크

Feed에서 RSS 를 가져왔으나 노션 데이터베이스에 같은 Guid 항목이 있어서 필터되어 다른 AI기사가 DB에 저장됨.

실행 완료:

<img src="../images/notion_guid_check.jpg" width="800"/>

<br>

노션 모듈에서 Guid 체크 output:

<img src="../images/notion_guid_check_output.png" width="500"/>

<br>

노션 데이터 베이스에 다른 기사 저장:

<img src="../images/notion_guid_check_table_result.png" width="800"/>

<br>

---


<!--

<font color="red">로그 종류 판단하시고 필요에 따라 정의 변경, 종류 변경 해주세요.</font>

## 접속 에러 알림
* 접속 시도시 실패한 기록 - 연결 실패(인터넷 에러, 잘못된 주소 등)

## 실행 로그 (재실행 로그)
* 워크플로우가 실행된 이력 (성공, 실패 여부 기록이 되면 더욱 좋습니다)
  * 실패 뒤에 실행기록이 쌓이면 그것이 재실행이기 때문에 성공,실패 기록이 들어가면 유리함

-->

# ⚠️ 에러 처리 정책 및 선택 이유

본 워크플로우는 **1) 즉각 조치가 필요한 핵심 모듈(RSS, OpenAI, Notion)은 이메일 알림 및 재시도(Retry)** 정책을 적용하고, **2) 시스템 연속성이 중요한 모듈(HTTP, JSON)은 구글 시트 기록 및 스킵(Skip)** 정책을 적용하는 등 에러의 치명도에 따라 대응 프로세스를 다각화하였다.

또한, 라우터(Router)와 필터(Filter)를 적재적소에 배치하여 문법적 오류(System Error)와 데이터 정제 오류(Business Logic Issue)를 분리 대응하도록 구현하였다.

## ⚙️ 에러 처리 정책 및 선택 이유 일람표 (시스템 에러 대응)

<img src="../images/measure_system_error.jpg" />

| 분류 | 대상 모듈 | 에러/이슈 상황 및 테스트 케이스 | 대응 정책 (Error Handling) | 대응 방식 선택 이유 (실무적 근거) |
| --- | --- | --- | --- | --- |
| **시스템 에러** | **[RSS]** | RSS 피드 URL이 존재하지 않을 경우  (예: `https://.../fe`와 같이 잘못된 주소) | 관리자 **Email 즉시 알림** 발송 | 데이터 수집의 시작점인 RSS 모듈이 작동하지 않으면 전체 파이프라인이 중단되므로, 관리자가 인지하고 즉각 조치할 수 있도록 이메일 알림을 설정함. |
| **시스템 에러** | **[Notion]** (1, 2차) | Data Source ID(데이터베이스 ID)가 존재하지 않아 노션 연동에 실패한 경우 | 관리자 **Email 알림** 발송 후 **`Retry(재시도)`** | 노션 API 장애나 일시적인 서버 불안정으로 인한 데이터 유실을 막기 위해 관리자에게 알림을 보냄과 동시에 재시도를 수행하여 데이터 적재 안정성을 확보함. |
| **시스템 에러** | **[OpenAI]** (1, 2차) | 존재하지 않는 GPT 모델명을 강제로 입력한 경우 (예: `gpt-fake-model-12345`) | 관리자 **Email 알림** 발송 후 **`Retry(재시도)`** | LLM API 에러는 일시적인 할당량 초과나 타임아웃일 확률이 높으므로, 즉시 종료하기보다 시간 차를 두고 재시도하여 뉴스 요약 업무의 연속성을 유지함. |
| **시스템 에러** | **[HTTP]** | 뉴스 원문 스크래핑 시 잘못된 URL이 전달된 경우 (예: [https://doesntexist.com](https://doesntexist.com)) | Google Sheets에 로그 생성 후 **`Skip`** 처리 | 특정 뉴스 사이트의 서버 다운이나 링크 만료는 인프라 전체의 치명적인 에러가 아니므로, 시스템을 중단시키지 않고 구글 시트에 이력을 남김. |
| **시스템 에러** | **[Parse JSON]** | OpenAI가 반환한 결과물이 올바른 JSON 형식이 아닐 경우 (예: `not-json-you-want` 입력) | Google Sheets에 로그 생성 후 **`Skip`** 처리 | 완벽한 구조를 갖추지 못한 문자열이 들어왔을 때 모듈이 크래시(Crash)되는 것을 방지하고, 에러 원인을 구글 시트에 기록하여 디버깅을 용이하게 함. |

---

## 📊 비즈니스 로직 및 예외 처리 일람표 (데이터/라우터 흐름 대응)

시스템 자체의 에러는 아니지만, 비즈니스 요구사항을 충족하지 못하는 '데이터 이슈' 및 '필터링'을 처리하기 위한 정책 설계이다.

<img src="../images/measure_other_issue.jpg" />

| 분류 | 적용 구간 | 처리 내용 및 테스트 케이스 | 대응 정책 (Routing / Filtering) | 대응 방식 선택 이유 (실무적 근거) |
| --- | --- | --- | --- | --- |
| **중복 방지** | **[Notion 적재 전]** | RSS 피드에서 가져온 기사의 고유 식별자(`Guid`)가 노션 DB에 이미 존재하는 경우 | Filter를 통해 중복 데이터는 제외하고 **새로운 기사만 선택적 적재** | 매일 스케줄러가 돌아갈 때 동일한 기사가 노션에 중복으로 쌓여 데이터베이스가 오염되는 것을 방지하고 효율적인 용량 관리를 도모함. |
| **데이터 필터링** | **[AI 뉴스 필터링]** | 수집된 뉴스 중 AI/테크 관련 핵심 키워드가 전혀 포함되지 않은 경우 | Router 우회 경로를 통해 **Google Sheets(issue_log)에 기록** | 노션 DB에는 양질의 AI 뉴스만 저장해야 하므로 필터링하고 그날은 AI 관련 뉴스가 없었음을 숙지하기 위함. |
| **데이터 필터링** | **[HTTP ➡️ Text Parser]** | HTTP 응답 코드(Status Code)가 성공 범위가 아닐 때 (필터 조건: 200~299 아님) | 다음 모듈(`Text Parser`)로 가지 못하도록 **진입 차단** | 비정상적인 웹페이지(404 Not Found, 500 Error 등)의 에러 텍스트가 파서로 넘어가 토큰을 낭비하거나 엉뚱한 요약을 생성하는 것을 차단함. |
| **데이터 이슈** | **[Parse JSON 누락]** | JSON 문법은 올바르나 필수 데이터(`Summary`)가 누락된 경우 | Router의 **Fallback 경로**를 통해 **Google Sheets에 기록** | 구조는 정상이나 불완전한 데이터가 노션 DB에 저장되는 것을 방지(데이터 정제)하고, 누락 이력을 추적하여 AI 프롬프트를 보완하기 위함. |

---