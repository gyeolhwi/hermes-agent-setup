# Hermes Agent 설정 가이드

Hermes Agent를 **내 PC, WSL2 또는 원격 Ubuntu 서버**에 설치하고, 필요한 기능만 연결하기 위한 실전 가이드입니다.

Hermes는 AI 모델과 도구를 연결해 대화·조사·문서 작업·파일 작업·반복 작업 등을 돕는 개인 또는 팀용 AI 에이전트입니다. Telegram·Slack 같은 메신저에서 사용할 수도 있고, 터미널에서만 사용할 수도 있습니다.

## 이 저장소가 해주는 일

```text
설치 위치 결정 → Hermes 설치 → AI 모델 연결 → 필요한 연동 선택 → 실제 동작 확인
```

## 먼저 결정할 것

| 내가 원하는 방식 | 먼저 읽을 문서 |
|---|---|
| 내 PC에서 바로 사용 | [빠른 시작](docs/quick-start.md) |
| WSL2 Ubuntu에서 사용 | [빠른 시작](docs/quick-start.md) |
| AI에게 현재 PC 또는 원격 서버 세팅을 맡기기 | [AI에게 세팅 맡기기](docs/ai-assisted-setup.md) |
| 원격 Ubuntu 서버에 항상 켜기 | [원격 서버 설정](docs/remote-server.md) |
| Slack 봇을 처음 만들기 | [Slack 봇 연결](docs/slack-bot-setup.md) |
| Telegram·메일 등 선택 연동 | [선택 연동](docs/optional-integrations.md) |

## 시작 전, 내가 준비할 것

- 사용할 장소: 내 PC / WSL2 / 원격 Ubuntu 서버
- 사용할 AI 모델 계정: ChatGPT·Claude·Gemini·Nous Portal 등 중 하나
- 메신저를 연결한다면: 해당 서비스의 앱 생성 또는 로그인 권한
- 원격 서버라면: 비밀번호 없이 접속되는 SSH 별칭

토큰·비밀번호·SSH 개인키는 이 저장소나 채팅에 넣지 않습니다. 설정 중 해당 값이 필요하면 **본인이 직접 입력**합니다.

## 가장 짧은 설치 흐름

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
source ~/.bashrc
hermes setup
hermes doctor
hermes
```

`hermes setup`에서 모델을 연결합니다. 설치가 끝난 뒤에는 `hermes`를 실행해 바로 대화할 수 있습니다.

## 선택 기능

| 기능 | 언제 연결하면 좋은가 |
|---|---|
| Telegram | 휴대폰에서 개인 AI 비서처럼 쓰고 싶을 때 |
| Slack | 팀 채널 또는 업무용 봇으로 쓰고 싶을 때 |
| IMAP 메일 | 메일함 조회·정리·초안을 도울 때 |
| Gateway + systemd | 원격 서버에서 메신저 봇을 계속 켜둘 때 |
| Skills | 반복하는 업무 절차를 Hermes가 기억하게 할 때 |

연결 방법과 확인 기준은 [선택 연동](docs/optional-integrations.md)에 있습니다.

## AI에게 설치를 맡기기

현재 컴퓨터에 설치하는 경우와 SSH 원격 서버에 설치하는 경우 모두 [AI에게 세팅 맡기기](docs/ai-assisted-setup.md)의 안내 문구를 그대로 사용할 수 있습니다.

AI는 설치·점검·검증을 돕고, 사용자는 비밀번호·토큰·OAuth 로그인만 직접 처리합니다.

## 보안 원칙

- 비밀값은 `~/.hermes/.env`, `~/.hermes/auth.json`, `~/.secrets/`처럼 권한이 제한된 위치에만 둡니다.
- `.env`, `auth.json`, SSH 키, 메일 비밀번호를 Git에 커밋하지 않습니다.
- 메신저·메일·파일 수정·서버 명령은 연결한 계정과 권한 범위에서만 동작합니다.
- “설정됨” 표시만 믿지 말고, 모델 응답·메시지 수신·메일함 조회처럼 실제 동작으로 확인합니다.

## 공식 문서

- [Hermes 설치](https://hermes-agent.nousresearch.com/docs/getting-started/installation)
- [AI 모델 공급자](https://hermes-agent.nousresearch.com/docs/integrations/providers)
- [메시징 Gateway](https://hermes-agent.nousresearch.com/docs/user-guide/messaging)
- [Slack 연결](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/slack)
