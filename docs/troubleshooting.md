# Troubleshooting

이 문서는 작업 중 실제로 겪었던 오류와 해결 방법을 정리한 문서입니다.

## Docker API 연결 실패

오류:

```text
failed to connect to the docker API at npipe:////./pipe/dockerDesktopLinuxEngine
```

원인:

- Docker Desktop이 실행되지 않음
- Docker Engine이 아직 준비되지 않음
- WSL2 업데이트 중이거나 WSL2 상태가 불안정함

해결:

```text
1. Docker Desktop 실행
2. Docker Desktop is running 상태 확인
3. PowerShell에서 docker version 확인
4. Client와 Server가 모두 보이면 다시 n8n 실행
```

## Windows에서 Docker 명령 줄바꿈 실패

증상:

```text
'--name'은 내부 또는 외부 명령이 아닙니다.
'-p'은 내부 또는 외부 명령이 아닙니다.
```

원인:

- Linux/macOS 스타일 줄바꿈 문자 `\`를 Windows cmd/PowerShell에서 그대로 사용함

해결:

Docker run 명령을 한 줄로 실행하거나 PowerShell 백틱을 사용합니다.

권장:

```powershell
docker run -it --rm --name n8n -p 5678:5678 -e GENERIC_TIMEZONE="Asia/Seoul" -e TZ="Asia/Seoul" -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

## ngrok 명령어를 찾을 수 없음

오류:

```text
ngrok : 'ngrok' 용어가 cmdlet, 함수, 스크립트 파일 또는 실행할 수 있는 프로그램 이름으로 인식되지 않습니다.
```

원인:

- ngrok이 설치되지 않았거나 PATH에 등록되지 않음

해결:

```powershell
winget install ngrok.ngrok
```

또는 직접 다운로드 후 실행:

```powershell
C:\ngrok\ngrok.exe http 5678
```

## ngrok authtoken 오류

오류:

```text
authentication failed: Usage of ngrok requires a verified account and authtoken.
ERR_NGROK_4018
```

원인:

- ngrok 계정 authtoken 미등록

해결:

```powershell
ngrok config add-authtoken 본인_토큰
ngrok http 5678
```

## ngrok 버전이 너무 오래됨

오류:

```text
Your ngrok-agent version "3.3.1" is too old.
The minimum supported agent version for your account is "3.20.0".
ERR_NGROK_121
```

원인:

- 설치된 ngrok 버전이 계정 최소 요구 버전보다 낮음

해결:

```powershell
ngrok update
```

또는:

```powershell
winget upgrade ngrok.ngrok
```

## n8n Webhook에서 Respond 노드 없음

오류:

```text
No Respond to Webhook node found in the workflow
```

원인:

- Webhook 노드가 `Using Respond to Webhook Node`로 설정되어 있는데 workflow 안에 `Respond to Webhook` 노드가 없음

해결:

```text
Webhook
→ Respond to Webhook
```

구조를 추가합니다.

## Respond to Webhook JSON 오류

오류:

```text
Invalid JSON in 'Response Body' field
```

원인:

- Response Body에 유효하지 않은 JSON 입력
- 줄바꿈이 포함된 값을 따옴표 안에 직접 넣음

해결:

테스트용 고정 JSON:

```json
{
  "response_type": "ephemeral",
  "text": "n8n Slack 연결 테스트 성공"
}
```

동적 text를 넣을 때는 n8n 필드 입력 방식 또는 `JSON.stringify`를 사용합니다.

## HTTP Request 상세 URL 앞에 =가 붙는 오류

오류:

```text
Invalid URL: =https://www.jobkorea.co.kr/Recruit/GI_Read/...
URL must start with "http" or "https".
```

원인:

- URL 입력칸에서 `={{ $json.url }}`가 문자열로 처리됨

해결:

```text
좋음: {{ $json.url }}
나쁨: ={{ $json.url }}
```

표현식 모드가 필요하면 n8n의 expression 모드에서 `$json.url`을 사용합니다.

## OpenAI quota/rate limit 오류

오류:

```text
The service is receiving too many requests from you
You exceeded your current quota, please check your plan and billing details.
```

원인:

- OpenAI API 결제 수단 미등록
- 무료 크레딧 만료
- 사용량 한도 초과
- 동시에 너무 많은 요청 발생

해결:

```text
1. OpenAI Platform Billing 확인
2. Usage / Limits 확인
3. 공고 개수를 .slice(0, 1)로 제한
4. detailText 길이를 3000자 이하로 제한
```

## Slack에서 /jobs 실패 메시지

증상:

```text
{reason} 오류가 발생해 /jobs에 실패했습니다.
```

주요 원인:

- Slack slash command 3초 제한 초과
- n8n workflow 내부 오류
- Respond to Webhook 응답 형식 오류

해결:

기존 구조:

```text
Webhook
→ 잡코리아 조회
→ OpenAI 요약
→ Respond to Webhook
```

개선 구조:

```text
Webhook
→ 즉시 Respond to Webhook
→ 잡코리아 조회
→ OpenAI 요약
→ Slack response_url로 결과 전송
```

## Slack에 [object Object]가 출력됨

증상:

```text
[object Object]
```

원인:

- OpenAI 노드 응답이 문자열이 아니라 객체인데 그대로 Slack text에 넣음

해결:

- OpenAI 응답에서 `text`, `content`, `message`, `output`, `response`, `result` 등을 재귀적으로 탐색해 문자열을 추출합니다.
- `docs/workflow-nodes.md`의 `Code: Build Slack Result` 코드를 사용합니다.

## ngrok 로그는 찍히는데 Slack 응답이 없음

증상:

```text
POST /webhook/slack/jobkorea-jobs
```

하지만 Slack에서 실패 처리됨.

원인:

- n8n이 응답을 완료하지 못함
- AI 요약이 느려 Slack 3초 제한 초과

해결:

- 즉시 응답 후 `response_url`로 후속 결과를 보내는 구조로 변경합니다.
