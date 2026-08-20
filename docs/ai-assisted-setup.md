# AI에게 Hermes 세팅 맡기기

이 저장소를 내려받은 뒤, Claude Code·Codex·Hermes 같은 AI 코딩 에이전트에게 설치를 맡길 수 있습니다.

AI는 설치·점검·문서 읽기·검증을 맡고, 사용자는 **로그인·토큰·비밀번호 입력만 직접** 하면 됩니다.

## 1. 저장소 받기

```bash
git clone <이 저장소의 HTTPS 주소>
cd hermes-agent-setup
```

## 2. 현재 컴퓨터에 설치할 때

AI 에이전트를 이 저장소 폴더에서 열고 아래처럼 요청합니다.

```text
이 저장소의 README.md, AGENTS.md, docs/quick-start.md를 읽고
현재 컴퓨터에 Hermes를 설치해줘.
먼저 기존 Hermes와 시스템 상태를 읽기 전용으로 점검하고,
내가 필요한 기능만 설정해줘.
비밀번호·토큰·OAuth 로그인 입력이 필요하면 멈춰서 내가 직접 하게 해줘.
마지막에는 hermes doctor와 실제 모델 응답으로 확인해줘.
```

## 3. 원격 서버에 설치할 때

먼저 작업자 컴퓨터에서 SSH 별칭을 만듭니다.

```text
Host my-hermes
    HostName <서버 주소>
    User <서버 계정>
    IdentityFile ~/.ssh/<개인키 파일>
    IdentitiesOnly yes
```

아래 명령이 비밀번호 없이 성공해야 합니다.

```bash
ssh my-hermes 'whoami; hostname'
```

그 뒤 AI 에이전트에게 요청합니다.

```text
ssh my-hermes로 접속해 이 저장소 기준으로 Hermes를 설치해줘.
docs/remote-server.md를 먼저 읽고 서버 상태를 점검해줘.
토큰·비밀번호·OAuth 로그인처럼 내가 직접 입력할 내용이 필요하면
tmux 세션을 만들어 접속 방법과 입력 안내를 알려줘.
입력이 끝나면 Gateway와 실제 모델 응답까지 검증해줘.
```

AI가 비밀값 입력 단계에서 멈추면, 안내받은 명령으로 tmux 화면에 붙습니다.

```bash
ssh -t my-hermes 'tmux attach -t setup'
```

입력을 마친 뒤 `Ctrl+b`를 누르고 손을 뗀 다음 `d`를 누르면 tmux 화면에서 나올 수 있습니다.

## 4. 사용자가 직접 해야 하는 일

| 항목 | 이유 |
|---|---|
| 모델 계정 로그인 | OAuth·구독·API 키는 본인 계정 권한이 필요함 |
| Slack·Telegram 토큰 입력 | 비밀값을 AI 대화나 Git에 남기지 않기 위함 |
| 메일 비밀번호 입력 | 메일함 권한을 본인이 통제하기 위함 |
| 실제 메신저 테스트 메시지 전송 | AI가 아닌 실제 사용자가 수신 위치를 확인하기 위함 |

## 5. 완료로 볼 기준

```bash
hermes doctor
hermes gateway status
```

그리고 모델에 한 문장을 보내 응답을 받고, 연결한 메신저가 있다면 실제 테스트 메시지에도 응답이 와야 합니다.
