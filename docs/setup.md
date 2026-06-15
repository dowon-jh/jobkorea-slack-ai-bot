# Setup Guide

이 문서는 로컬 PC에서 n8n 기반 잡코리아 Slack 봇을 실행하는 절차입니다.

## 1. Docker Desktop 실행

Docker Desktop을 먼저 실행한 뒤 PowerShell에서 확인합니다.

```powershell
docker version
```

`Client`와 `Server`가 모두 나오면 정상입니다.

## 2. n8n 실행

최초 1회 volume을 생성합니다.

```powershell
docker volume create n8n_data
```

n8n을 실행합니다.

```powershell
docker run -it --rm --name n8n -p 5678:5678 -e GENERIC_TIMEZONE="Asia/Seoul" -e TZ="Asia/Seoul" -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

브라우저에서 접속합니다.

```text
http://localhost:5678
```

## 3. ngrok 실행

새 PowerShell 창에서 실행합니다.

```powershell
ngrok http 5678
```

출력 예시:

```text
Forwarding https://your-ngrok-domain.ngrok-free.dev -> http://localhost:5678
```

이 HTTPS 주소를 Slack Slash Command Request URL에 사용합니다.

## 4. Slack Slash Command 설정

Slack API 페이지에서 앱을 만들고 Slash Command를 추가합니다.

```text
Command: /jobs
Request URL: https://your-ngrok-domain.ngrok-free.dev/webhook/slack/jobkorea-jobs
Short Description: 잡코리아 채용공고 AI 요약
Usage Hint: 데이터엔지니어
```

설정 후 `Install to Workspace` 또는 `Reinstall to Workspace`를 진행합니다.

## 5. OpenAI Credential 설정

n8n의 OpenAI 노드에서 API Key credential을 추가합니다.

```text
Credential type: OpenAI API
API Key: OpenAI Platform에서 발급한 secret key
```

요약용 모델은 비용과 속도를 고려해 가벼운 모델을 권장합니다.

```text
gpt-4o-mini
gpt-4.1-mini
```

## 6. 실행 순서

매번 사용할 때 기본 순서는 다음과 같습니다.

```text
1. Docker Desktop 실행
2. n8n Docker container 실행
3. ngrok http 5678 실행
4. Slack Request URL이 현재 ngrok 주소인지 확인
5. n8n workflow Published 상태 확인
6. Slack에서 /jobs 실행
```

## 7. 무료 ngrok 사용 시 주의사항

무료 ngrok은 주소가 바뀔 수 있습니다. 주소가 바뀌면 Slack Slash Command의 Request URL도 수정해야 합니다.
