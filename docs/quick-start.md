# 빠른 시작

이 문서는 **내 PC 또는 WSL2 Ubuntu에서 Hermes를 처음 실행**하는 가장 짧은 경로입니다.

## 내가 할 일

| 단계 | 내가 준비하거나 선택할 것 |
|---|---|
| 1 | 설치할 환경: Windows·macOS·Linux·WSL2 중 하나 |
| 2 | AI 모델 계정: ChatGPT, Claude, Gemini, Nous Portal 등 중 하나 |
| 3 | 설치 후 사용할 방식: 터미널만 / Telegram / Slack / 메일 |

## 1. 설치

### Linux · macOS · WSL2

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
source ~/.bashrc
```

### Windows

PowerShell에서 실행합니다.

```powershell
iex (irm https://hermes-agent.nousresearch.com/install.ps1)
```

Windows·macOS에서 데스크톱 앱도 함께 쓰고 싶다면 [Hermes Desktop 설치 프로그램](https://hermes-agent.nousresearch.com/)을 이용하는 편이 편합니다.

## 2. AI 모델 연결

```bash
hermes setup
```

설정 화면에서 사용할 모델 공급자를 고릅니다. 토큰이나 브라우저 로그인이 필요하면 **본인이 직접** 진행합니다.

처음에는 한 모델만 연결하면 충분합니다. 나중에 아래 명령으로 바꿀 수 있습니다.

```bash
hermes model
```

## 3. 정상 설치 확인

```bash
hermes doctor
hermes
```

`hermes`가 열리면 간단히 한 문장을 물어봅니다.

```text
오늘 할 일을 세 가지로 정리해줘.
```

응답이 오면 기본 설치는 끝입니다.

## 다음에 할 일

| 원하면 | 이동 |
|---|---|
| 휴대폰·팀 채팅에서 쓰기 | [선택 연동](optional-integrations.md) |
| 원격 서버에서 계속 실행하기 | [원격 서버 설정](remote-server.md) |
| 반복 작업을 기억시키기 | Hermes의 Skills 기능과 공식 문서 참고 |

## 문제가 생기면

```bash
hermes doctor
hermes gateway status
```

명령 결과에 비밀값이 보인다면 공유하기 전에 지웁니다.
