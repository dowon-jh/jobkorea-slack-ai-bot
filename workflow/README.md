# Workflow Export

이 폴더에는 n8n workflow export JSON을 저장합니다.

## Export 방법

n8n workflow 화면에서 다음 순서로 내보냅니다.

```text
Workflow 화면
→ 우측 상단 메뉴
→ Download / Export workflow
→ JSON 저장
```

권장 파일명:

```text
jobkorea-slack-ai-bot.workflow.json
```

## 주의사항

export된 JSON에 아래 값이 포함되어 있지 않은지 확인하세요.

```text
OpenAI API Key
Slack token
ngrok authtoken
실제 Slack response_url
개인 ngrok 고정 도메인
```

민감값이 들어 있다면 제거한 뒤 GitHub에 올립니다.
