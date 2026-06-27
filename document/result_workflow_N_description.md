# 워크플로우 구조 (Make)

### 1. 스크린샷

[[워크플로우 링크]](https://eu1.make.com/public/shared-scenario/UTrlwkBAsGN/integration-rss-email)

<img src="../images/workflow.jpg"/>

---

# 각 단계별 역할과 연결 구조를 설명하는 문서

## 1. 워크플로우 흐름도

```text
▼ [1. 🌐 RSS 트리거] 
│   └─ (에러 발생 시) ➔ 📧 Send an Email
│
├── ⚡ [Filter 1] AI 관련 키워드 체크
│
▼ [2. 📝 Notion 중복 검사] (Guid 기준 조회)
│   └─ (에러 발생 시) ➔ 📧 Send an Email + 🔄 Retry 2회
│
├── ⚡ [Filter 2] Notion 검색 결과 (Total nr of bundles = 0) 일 때만 통과
│
▼ [3. 📑 Text Aggregator] (신규 AI 기사들을 하나의 텍스트로 합산)
│
▼ [4. 🔀 라우터 (Router) 분기]
    │
    ├─── [Path A] 금일 AI 기사가 존재하는 경우
    │     │
    │     ▼ [5. 🤖 OpenAI (선택)] 오늘의 Top 기사 URL 1개 선정
    │     │   └─ (에러 발생 시) ➔ 📧 Send an Email + 🔄 Retry 2회
    │     │
    │     ├── ⚡ [Filter 3] OpenAI 결과 URL에 'http'가 포함되면 통과
    │     │
    │     ▼ [6. 🌐 HTTP (크롤링)] 웹페이지 원시 HTML 소스 다운로드
    │     │   └─ (에러 발생 시) ➔ 📊 Google Sheets (오류 기록) [2차 에러 무시] + Skip
    │     │
    │     ├── ⚡ [Filter 4] HTTP Status Code가 200~299이면 통과
    │     │
    │     ▼ [7. 📝 Text Parser] HTML 태그 제거 및 순수 텍스트 추출
    │     │
    │     ▼ [8. 🤖 OpenAI (요약)] 기사 전문 분석 후 요약 및 JSON 출력
    │     │   └─ (에러 발생 시) ➔ 📧 Send an Email + 🔄 Retry 2회
    │     │
    │     ▼ [9. ⚙️ JSON 파싱] JSON 텍스트를 개별 데이터 변수로 분해
    │     │   └─ (에러 발생 시) ➔ 📊 Google Sheets (오류 기록) [2차 에러 무시] + Skip
    │     │
    │     ▼ [10. 🔀 라우터 (Router) 2차 분기]
    │         │
    │         ├─── [Path A-1] JSON 필수 데이터가 모두 존재하는 경우
    │         │     │
    │         │     ▼ [11. 📝 Notion 저장] DB 내 최종 저장 완료 ✨
    │         │         └─ (에러 발생 시) ➔ 📧 Send an Email + 🔄 Retry 2회
    │         │
    │         └─── [Path A-2] JSON 데이터 중 일부가 누락된 경우
    │               │
    │               ▼ [12. 📊 Google Sheets (로그 기록)] "데이터 누락 실패" 행 추가 [2차 에러 무시]
    │
    └─── [Path B] Fallback: 금일 AI 기사가 없는 경우
          │
          ▼ [13. 📊 Google Sheets (로그 기록)] "AI 관련 기사가 없습니다" 행 추가 [2차 에러 무시]
```


## 2. 워크플로우 단계별 설명

### [1단계]: 데이터 수집 및 1차 필터링

| 모듈명 | 모듈 설명 | 역할 | 단계 산출물 |
| --- | --- | --- | --- |
| **[1] RSS** - Watch RSS Feed items | TechCrunch 등 지정된 RSS 피드를 실시간/주기적으로 감시 | 새로운 뉴스 기사가 발행되면 데이터를 가져오는 트리거 역할 | 새 뉴스 기사 데이터 (Title, URL, Description, Guid 등) |
| **(Filter 1)** - AI 관련 기사 필터 | 기사 내용에 AI 관련 키워드가 포함되어 있는지 검사 | AI와 관련 없는 일반 테크 뉴스를 1차로 걸러냄 | 필터 조건을 충족한 AI 관련 기사 데이터 |

---

### [2단계]: 중복 제거 및 데이터 통합

| 모듈명 | 모듈 설명 | 역할 | 단계 산출물 |
| --- | --- | --- | --- |
| **[2] Notion** - Search Objects | Notion 데이터베이스에서 현재 기사의 `Guid`를 검색 | 이미 예전에 노션에 저장했던 중복 기사인지 확인. | Notion 검색 결과 (Bundle 수) |
| **(Filter 2)** - 중복 제거 필터 | Notion 검색 결과의 총 Bundle 수가 **0일 때만** 통과시킴 | 이미 노션에 있는 기사는 제외하고, **새로운 기사만** 다음 단계로 보냄 | 노션에 존재하지 않는 신규 AI 기사 데이터 |
| **[3] Tools** - Text Aggregator | 여러 개로 나뉜 신규 AI 기사들을 하나의 텍스트 템플릿으로 묶음 | 여러 번 실행될 OpenAI 호출을 1번으로 줄여 비용 절감 | 모든 신규 AI 기사가 누적된 하나의 통합 텍스트 변수 (`text`) |

---

### [3단계]: 조건별 경로 분기 (Router)

| 모듈명 | 모듈 설명 | 역할 | 단계 산출물 |
| --- | --- | --- | --- |
| **[4] Router** - 라우팅 분기 | 통합된 텍스트(`text`)의 존재 여부에 따라 경로를 나눔 | 오늘 처리할 AI 뉴스가 있는 경우와 없는 경우의 동작을 분기 | 조건에 따른 경로 배정 |
| **Path A Filter** - AI 기사 있음 | `text` 변수가 비어있지 않은 경우 진행 | 메인 요약 및 저장 프로세스 가동 | 통합 기사 텍스트 전달 |
| **Path B** - Fallback (기사 없음) | Fallback(Yes)으로 설정되어, Path A 조건에 맞지 않을 때 자동 실행 | **[Google Sheets - Add a Row]** 모듈로 이동하여 금일 날짜와 함께 로그 기록 | 구글 시트 내 *"AI 관련 기사가 없습니다"* 로그 행 추가 |

---

### [4단계]: 핵심 기사 선정 및 본문 크롤링 (Path A 진행 시)

| 모듈명 | 모듈 설명 | 역할 | 단계 산출물 |
| --- | --- | --- | --- |
| **[5] OpenAI** - Generate a completion | 통합된 기사 리스트 중 가장 중요도가 높은 기사 1개 선정 | AI가 가치 판단을 통해 오늘 꼭 읽어야 할 탑 기사의 URL만 딱 하나 추출함 | 선별된 기사의 URL |
| **(Filter 3)** - URL 유효성 필터 | OpenAI가 추출한 결과물에 실제 웹 주소가 포함되어 있는지 검사 | 잘못된 텍스트가 HTTP 모듈로 넘어가 에러가 나는 것을 사전에 방지 | `http` 키워드가 포함된 유효한 URL |
| **[6] HTTP** - Make a request | OpenAI가 선택한 기사의 URL 링크로 웹 요청을 보냄 | 해당 뉴스 웹페이지에 직접 방문하여 데이터를 긁어옴 | 웹페이지의 원시 HTML 소스 코드 |
| **(Filter 4)** - 정상 응답 필터 | HTTP 응답 상태 코드(Status Code)를 확인 | 정상적인 웹페이지 수집 성공 여부 판별 | `Status Code`가 200 이상 300 미만인 데이터 (200~299) |
| **[7] Text Parser** - HTML to text | 다운로드한 HTML 코드에서 텍스트만 추출 | `<script>`, `<div>` 등 불필요한 태그를 지우고 기사 본문 텍스트만 남김 | 기사 본문 텍스트 |

---

### [5단계]: AI 심층 요약 및 최종 저장

| 모듈명 | 모듈 설명 | 역할 | 단계 산출물 |
| --- | --- | --- | --- |
| **[8] OpenAI** - Generate a completion | 기사 본문을 기반으로 최대 3개의 불릿 포인트 요약 및 메타데이터 생성 | 구조화된 분석을 수행하고, 프로그래밍 처리가 쉽도록 **JSON 형태**로 결과를 출력 | Title, Summary, URL 등이 포함된 JSON 텍스트 |
| **[9] JSON** - Parse JSON | OpenAI가 출력한 JSON 텍스트를 구조화된 데이터로 파싱 | 텍스트 덩어리를 Notion 필드에 각각 맵핑할 수 있는 개별 변수로 변환 | 분리된 데이터 필드 (Title, Summary, URL 등) |

---

### [6단계]: 데이터 검증 분기 및 최종 저장

| 모듈명 | 모듈 설명 | 역할 | 단계 산출물 |
| --- | --- | --- | --- |
| **[10] Router** - 데이터 검증 라우터 | 파싱된 JSON 결과물에 필수 데이터가 누락되었는지 2차 검증 | 데이터가 비어있는 상태로 노션에 저장되어 DB가 오염되는 것을 방지 | 조건에 따른 경로 배정 |
| **Path A-1 Filter** - 데이터 완벽 통과 | 필수 변수들(Title, Summary, URL, Date, Emotion, Guid)이 모두 존재하고 비어있지 않은 경우 (`Exist`) | 최종 저장 단계로 데이터 이행 | 검증이 완료된 완벽한 기사 변수 |
| **[11] Notion** - Create a Database Item | 파싱된 데이터를 Notion DB의 각 속성(Column)에 맞게 입력 | 대시보드에 최종 최적화된 형태로 오늘의 AI 뉴스를 저장 | **노션 데이터베이스 새 행 생성 완료** |
| **Path A-2 Filter** - 데이터 일부 누락 | 필수 변수 중 하나라도 누락되거나 비어있는 경우 (`Not Exist`) | 에러 로그 기록 단계로 분기 | 누락이 발생한 원본 기사 URL 정보 |
| **[12] Google Sheets** - 로그 기록 | 데이터 누락으로 인해 노션 등록에 실패했음을 구글 시트에 기록 | 누락된 기사를 관리자가 모니터링하고 추후 수동 조치할 수 있도록 유도 | 구글 시트 내 *"Parse JSON 데이터 누락 실패"* 로그 행 추가 |

---

### [에러 처리]

에러 및 예외 처리에 관해 더 자세한 사항은 [[여기]](./result_error_details.md)를 참조하길 권한다.

* **🚨 에러 핸들러 (Error Handler):** 
  * RSS 피드 연결 실패 등 에러 발생 시 **[Email - Send an Email]** 모듈이 실행되어 관리자에게 에러 메시지 알림을 전송.
  * Notion/OpenAI 에러 시: 외부 서버 연동 에러(Timeout, API 한도 초과, 인증 만료 등) 발생 시, 관리자에게 상황을 공유하는 **[Email - Send an Email]** 및 [Retry 2회] 이 실행되어 이메일 알림 전송 후 재시도. 이를 통해 일시적인 트래픽 폭주나 네트워크 오류로 인해 전체 자동화가 멈추는 현상을 방지. 에러 알림(Email) 모듈에서 발생하는 2차 에러는 의도적으로 무시.  
  * HTTP 크롤링 에러 시: 웹사이트 차단 등으로 크롤링 실패 시, 전체 자동화가 멈추지 않도록 [Google Sheets - Add a Row] 에러 핸들러를 연결하여 오류 상황을 기록한 뒤, 해당 번들을 Skip.
  * JSON 파싱 에러 시: OpenAI의 출력 규격 리턴 오류로 파싱 실패 시, **[Google Sheets - Add a Row]** 에러 핸들러를 통해 로그를 남기고 Skip.

* **⛔ 2차 에러 처리:** 에러 핸들러 라인 내부의 Email과 Google sheets 모듈 자체에서 나는 에러는 시스템의 복잡도 감소를 위해 의도적으로 무시(Ignore)하도록 설계되었고 Make의 시나리오 설정(Scenario settings)에서 **Store incomplete executions = Yes**로 지정하여 '미완료 실행(Incomplete Execution)' 보관함에 저장되어 이메일 알림을 받고 추후 수동 복구가 가능하도록 2중 안전장치를 마련.