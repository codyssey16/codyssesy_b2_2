# 데이터베이스 구조 정의 및 데이터 예시

* **데이터베이스 : 노션 (Notion)**

## 1. 뉴스 Feed 테이블 속성

노션 DB 예시: [[링크]](https://app.notion.com/p/388b40a9099b803a9ebaf8c67768bcdf?v=388b40a9099b8097aad8000c3e7a5089) (접근 권한 요청 필요)

노션 DB 데이터베이스 페이지(published): [[링크]](https://rattle-sloop-646.notion.site/38a50b6922578084b15ae8d314fecf78?v=38a50b69225780009df2000cd656d866) (실제 Make 워크플로우에 연동된 DB)

스키마:

| 컬럼명 | 타입 | 설명 |
| :-- | :--:  | --: |
| Title  | text  | 기사 주제 |
| Summary  | text  | 3줄 요약 |
| URL  | URL  | URL 정보 
| Date  | datetime  | 기사 등록시각 |  
| Emotion  | text  | enum(긍정,부정)  |  
| Guid  | text  | Guid 고유키값 |  


## 2. 데이터 예시

테이블:

<img src="../images/success_notion_table.jpg" />

(시간: UDT time)

페이지:

<img src="../images/success_notion_page.jpg" width="500" />

(시간: UDT time)