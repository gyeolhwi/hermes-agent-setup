# Slack 봇 연결하기

이 문서는 Hermes를 Slack의 **팀용 AI 봇**으로 연결하는 안내입니다. 처음에는 과정이 많아 보이지만, 아래 순서대로 하면 됩니다.

```text
Hermes용 Slack 설정 생성 → Slack 앱 만들기 → 토큰 2개 발급 → Hermes에 직접 입력 → 실제 메시지로 확인
```

## 내가 준비할 것

| 준비물 | 이유 |
|---|---|
| Slack 앱을 만들 수 있는 워크스페이스 권한 | 앱 생성·설치에 필요 |
| 내 Slack Member ID | 봇을 사용할 사람을 제한할 때 필요 |
| 테스트할 채널 | 봇 초대와 멘션 테스트에 필요 |
| Hermes가 설치된 PC 또는 서버 | Gateway가 Slack과 계속 연결됨 |

토큰·비밀번호는 이 문서, Git, AI 채팅에 붙여 넣지 않습니다. 발급된 토큰은 **본인이 Hermes 설정 화면에 직접 입력**합니다.

## 가장 쉬운 방법: Hermes Manifest 사용

Hermes가 필요한 Slack 권한·이벤트·명령을 담은 Manifest를 만들어 줍니다. Slack 설정을 하나씩 빠뜨릴 가능성이 줄어드므로 이 방법을 권장합니다.

### 1. Hermes에서 Manifest 만들기

```bash
hermes slack manifest --agent-view --write
```

명령은 `~/.hermes/slack-manifest.json`에 설정 파일을 만듭니다.

> `--agent-view`는 새 Slack 앱에 권장되는 Agent 메시징 화면을 사용합니다. Slack에서 적용한 뒤에는 기존 Assistant 화면으로 되돌릴 수 없으므로, 새 앱에서 사용하세요.

### 2. Slack에서 앱 만들기

1. [Slack API 앱 관리](https://api.slack.com/apps)를 엽니다.
2. **Create New App** → **From an app manifest**를 선택합니다.
3. 사용할 Slack 워크스페이스를 고릅니다.
4. `~/.hermes/slack-manifest.json`의 내용을 붙여 넣고 **Create**를 누릅니다.
5. 앱을 만든 뒤 **Install App to Workspace**에서 설치를 승인합니다.

Manifest를 사용해도 아래의 토큰·DM·채널 테스트 단계는 반드시 진행합니다.

## 꼭 구분할 두 토큰

| 토큰 | 모양 | 하는 일 |
|---|---|---|
| Bot User OAuth Token | `xoxb-...` | 봇이 Slack 메시지를 읽고 답장함 |
| App-Level Token | `xapp-...` | Hermes Gateway가 Socket Mode 연결을 유지함 |

두 토큰은 서로 대체할 수 없습니다. `xapp-...` 하나만 있어 연결이 보이더라도 봇이 실제로 답장할 수 있는 것은 아닙니다.

### App-Level Token 만들기

Slack 앱 화면에서 다음으로 이동합니다.

```text
Settings → Socket Mode → Enable Socket Mode: ON
```

화면 안내에 따라 App-Level Token을 만들고 `connections:write` 권한을 선택합니다. 이 값이 `xapp-...` 토큰입니다.

**Socket Mode에서는 공개 Request URL이 필요하지 않습니다.** Hermes가 Slack에 WebSocket으로 직접 연결하므로, 노트북·WSL2·사설 서버에서도 사용할 수 있습니다.

### Bot Token 받기

Slack 앱 화면에서 다음으로 이동합니다.

```text
Settings → Install App → Install to Workspace → Allow
```

설치 뒤 보이는 `xoxb-...` 값이 Bot User OAuth Token입니다.

권한이나 이벤트를 나중에 바꿨다면 반드시 **Reinstall to Workspace**를 다시 실행합니다. 재설치 뒤 Bot Token이 새로 표시되면, Hermes에도 새 값을 다시 입력해야 합니다.

## Manifest를 쓰지 않을 때 확인할 항목

직접 Slack 앱을 구성했다면 아래 항목을 빠뜨리지 않았는지 확인합니다. 모두 Slack의 **Bot Token Scopes**에 넣습니다. User Token Scopes가 아닙니다.

```text
chat:write
app_mentions:read
channels:history, channels:read
groups:history              # 비공개 채널을 쓸 때
im:history, im:read, im:write
mpim:history, mpim:read     # 그룹 DM을 쓸 때
users:read
files:read, files:write     # 첨부 파일을 다룰 때
```

이벤트는 **Features → Event Subscriptions**에서 켜고, 필요한 Bot Event를 추가합니다.

```text
message.im       # 1:1 DM
message.mpim     # 그룹 DM
message.channels # 공개 채널
message.groups   # 비공개 채널
app_mention      # 채널 @멘션
```

DM을 사용한다면 **Features → App Home → Messages Tab**도 켠 뒤, “Allow users to send Slash commands and messages from the messages tab”을 선택해야 합니다. 이 설정이 없으면 DM 전송 자체가 막힙니다.

## Hermes에 연결하기

다음 명령으로 설정 화면을 엽니다.

```bash
hermes gateway setup
```

Slack을 선택하고 아래 값만 직접 입력합니다.

| Hermes 항목 | 넣을 값 |
|---|---|
| Bot Token | `xoxb-...` Bot User OAuth Token |
| App Token | `xapp-...` App-Level Token |
| Allowed Users | 내 Slack **Member ID** (`U...`) |
| Home Channel | 예약 메시지를 받을 실제 Slack **Channel ID** (`C...`), 선택 사항 |

**Member ID, Bot ID, Channel ID는 서로 다릅니다.**

- `Allowed Users`에는 봇 ID나 표시 이름이 아니라 **사람의 Member ID**를 넣습니다.
- Home Channel에는 DM ID가 아니라 실제 채널 ID를 넣습니다.
- Member ID는 Slack 프로필 → **View full profile** → **⋮** → **Copy member ID**에서 복사합니다.

설정 뒤 Gateway를 시작하거나, 이미 실행 중이면 설정을 반영합니다.

```bash
hermes gateway install
hermes gateway status
```

## Slack에서 실제로 확인하기

1. 테스트 채널에 봇을 초대합니다.

   ```text
   /invite @봇이름
   ```

2. 채널에서 봇을 멘션합니다.

   ```text
   @봇이름 안녕, Hermes 연결 테스트야.
   ```

3. DM도 쓸 예정이라면 Slack App Home에서 봇에게 메시지를 하나 보냅니다.
4. Hermes가 답장하면 연결 완료입니다.

동작 방식은 다음과 같습니다.

| 위치 | 기본 동작 |
|---|---|
| DM | 모든 메시지에 응답 |
| 채널 | 처음에는 `@봇이름` 멘션이 필요 |
| 이미 시작된 스레드 | 대화가 이어지는 동안에는 보통 멘션 없이도 응답 |

Slack은 스레드 답글 안에서 일반 `/명령`을 제한합니다. 스레드에서는 `/status` 대신 `!status`처럼 `!` 접두사를 사용하면 됩니다.

## 안 될 때, 이 순서로 확인

| 증상 | 먼저 볼 곳 | 해결 |
|---|---|---|
| `invalid_auth` | Bot Token | 새 `xoxb-...` 토큰을 입력하고 다시 확인 |
| `missing_scope` | Bot Token Scopes | 안내된 scope 추가 → **Reinstall to Workspace** → 새 토큰 반영 |
| DM을 보낼 수 없음 | App Home | Messages Tab과 메시지 허용 설정 켜기 |
| 채널에서 무반응 | Event Subscriptions / 채널 | `message.channels`, `app_mention` 확인 후 봇 초대·멘션 |
| 연결 로그만 있고 답장이 없음 | Gateway 로그 | 수신·응답·전송 단계가 모두 있는지 확인 |

```bash
hermes gateway status
hermes logs gateway -n 120
```

정상은 단순히 “connected”가 아니라, **Slack 메시지 수신 → Hermes 응답 생성 → Slack 전송 성공**까지 확인된 상태입니다.

## 공식 문서

- [Hermes 공식 Slack 안내](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/slack)
- [Slack API 앱 관리](https://api.slack.com/apps)
