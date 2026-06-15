# JobKorea Slack AI Bot

Slack slash command로 잡코리아 채용공고를 조회하고, n8n과 OpenAI를 이용해 공고 내용을 요약해서 보여주는 개인용 자동화 프로젝트입니다.

## 목표

Slack에서 `/jobs`를 입력하면 다음 흐름으로 동작합니다.

```text
Slack /jobs
→ n8n Webhook
→ 즉시 "조회 중입니다" 응답
→ 잡코리아 목록 페이지 조회
→ 공고 상세 페이지 조회
→ OpenAI로 자격요건/하게 될 업무 요약
→ Slack response_url로 결과 전송
```

## 결과 예시

```text
채용공고를 조회하고 AI로 요약하는 중입니다. 잠시만 기다려주세요.

잡코리아 AI 요약 채용공고

채용공고: 데이터 엔지니어 채용

자격요건:
- SQL 기반 데이터 처리 경험
- Python 활용 가능
- 데이터 파이프라인 이해

하게 될 업무:
- 데이터 수집 및 전처리
- ETL 파이프라인 운영
- 데이터 품질 점검
```

## 사용 기술

- n8n
- Docker Desktop
- ngrok
- Slack Slash Commands
- OpenAI API
- 잡코리아 공개 채용공고 목록/상세 페이지

## 저장소 구성

```text
jobkorea-slack-ai-bot/
  README.md
  .env.example
  .gitignore
  docs/
    setup.md
    workflow-nodes.md
    troubleshooting.md
  workflow/
    README.md
```

## 빠른 실행 순서

1. Docker Desktop 실행
2. n8n 실행
3. ngrok 실행
4. Slack Slash Command의 Request URL 확인
5. n8n workflow Publish
6. Slack에서 `/jobs` 실행

자세한 절차는 [docs/setup.md](docs/setup.md)를 참고하세요.

## 주의사항

- OpenAI API Key, Slack token, ngrok authtoken은 GitHub에 올리지 않습니다.
- 무료 ngrok은 실행할 때마다 URL이 바뀔 수 있습니다.
- Slack Slash Command는 3초 안에 응답해야 하므로, AI 요약 결과는 `response_url`로 나중에 전송하는 구조를 사용합니다.
- 잡코리아 페이지 구조가 바뀌면 HTML 파싱 코드 수정이 필요할 수 있습니다.
