# 선택 연동

기본 설치 뒤에는 원하는 기능만 추가합니다. 모두 연결할 필요는 없습니다.

## Telegram — 개인 AI 비서처럼 사용

**추천 대상:** 휴대폰에서 Hermes에게 바로 요청하고 싶은 경우

Hermes Gateway 설정에서 Telegram을 선택합니다. 처음 연결은 Hermes가 안내하는 QR 흐름을 우선 사용하면 토큰을 직접 다루는 과정을 줄일 수 있습니다.

```bash
hermes gateway setup
hermes gateway status
```

**확인:** Telegram에서 테스트 메시지를 보내 Hermes의 응답을 받습니다.

## Slack — 팀 채널에서 사용

**추천 대상:** 팀 채널에서 봇을 멘션하거나 스레드로 협업하고 싶은 경우

Slack은 Socket Mode를 사용하므로 서버를 공개 웹 주소로 열 필요가 없습니다. 다만 Slack App 생성, `xoxb-...` Bot Token과 `xapp-...` App Token 발급, Member ID 설정, 채널 초대가 필요합니다.

처음 연결한다면 [Slack 봇 연결 가이드](slack-bot-setup.md)를 순서대로 따르세요. Manifest 생성부터 DM·채널 테스트, 자주 생기는 `invalid_auth`·`missing_scope` 문제까지 다룹니다.

```bash
hermes slack manifest --agent-view --write
hermes gateway setup
```

## GitHub Issue ↔ Slack 웹훅 — 업무 기록과 대화 연결

**추천 대상:** GitHub Issue를 업무 기준 기록으로 쓰고, Issue별 Slack 스레드에서 진행 상황을 공유하고 싶은 경우

이 기능은 Hermes Gateway와 별도로 작은 Webhook Bridge 서비스를 운영합니다. GitHub의 `Issues`, `Issue comments` 이벤트를 서명 검증 후, 연결된 Slack 스레드에 전달합니다.

설치 전에는 [Issue ↔ Slack 웹훅 가이드](github-issue-slack-webhook.md)를 읽으세요. GitHub 저장소 관리자 권한, 고정 HTTPS 주소, Webhook Secret, 중복 방지용 연결 정보 저장이 필요합니다.

## IMAP 메일 — Himalaya로 메일함 다루기

**추천 대상:** 메일을 조회·분류·요약하거나 답장 초안을 만들고 싶은 경우

Hermes의 메신저 Gateway Email 설정과 Himalaya IMAP은 별개입니다. Himalaya는 터미널에서 메일함을 다루는 도구입니다.

1. Himalaya를 설치합니다.
2. IMAP·SMTP 계정을 설정합니다.
3. 비밀번호는 `~/.secrets/` 같은 권한 제한 파일에만 저장합니다.
4. 실제 연결은 메일함 목록으로 확인합니다.

```bash
himalaya account list
himalaya mailbox list
```

`himalaya mailbox list`가 성공해야 IMAP 연결이 실제로 된 것입니다. 메일을 보내거나 이동하는 작업은 먼저 초안·대상을 확인한 뒤 실행합니다.

## Gateway를 항상 켜기

**추천 대상:** 원격 서버에서 Telegram·Slack 봇을 계속 실행할 경우

```bash
hermes gateway install
hermes gateway status
```

Linux 서버에서는 다음도 확인합니다.

```bash
systemctl --user is-active hermes-gateway.service
loginctl show-user "$USER" -p Linger
```

## Dashboard

설정과 세션을 브라우저에서 보고 싶다면 실행합니다.

```bash
hermes dashboard
```

원격 서버에서는 Dashboard를 외부에 무심코 공개하지 않습니다. 기본 `127.0.0.1` 바인딩을 유지하고, 필요하면 SSH 터널이나 인증된 접근 방식을 사용합니다.

## 공통 보안 원칙

- 토큰·비밀번호·SSH 키는 Git·문서·메신저에 넣지 않습니다.
- 처음에는 허용 사용자와 채널을 최소한으로 설정합니다.
- 연결 뒤에는 “설정됨”이 아니라 실제 메시지·메일함 목록·응답으로 확인합니다.
