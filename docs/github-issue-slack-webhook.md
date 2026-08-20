# GitHub Issue ↔ Slack 웹훅 연결

이 문서는 GitHub Issue를 **업무의 기준 기록**으로 두고, 해당 Issue의 대화를 Slack 스레드에 연결하는 선택 기능입니다.

기본 Hermes·Slack 봇 설치에는 필요하지 않습니다. 팀이 GitHub Issue로 업무를 관리하고, Slack에서 진행 상황을 함께 보려 할 때만 추가합니다.

```text
Issue 생성 또는 변경
→ Issue별 Slack 부모글 1개 생성
→ 연결 정보 저장
→ GitHub Webhook 수신
→ 같은 Slack 스레드에 댓글·상태 변경 알림
```

## 이 구성이 해주는 일

| GitHub에서 일어난 일 | Slack에서 보이는 결과 |
|---|---|
| 새 Issue가 등록됨 | 해당 Issue 전용 부모글·스레드 1개 생성 |
| Issue 댓글이 추가됨 | 연결된 Slack 스레드에 댓글 전달 |
| Issue 상태·라벨이 변경됨 | 같은 스레드에 짧은 상태 변경 안내 |
| 완료 조건을 충족하고 Issue가 닫힘 | 부모글에 `✅` 반응과 완료 안내 |

이 자동화는 **알림과 기록 연결**을 위한 것입니다. Issue가 생성되거나 라벨이 바뀌었다고 해서 서버 파일 수정, 메일 발송, 외부 공개 같은 영향 있는 작업을 자동 실행하면 안 됩니다.

## 내가 준비할 것

| 준비물 | 이유 |
|---|---|
| GitHub 저장소 관리자 권한 | Webhook 등록에 필요 |
| 이미 연결된 Slack 봇 | 부모글·스레드 답변에 사용 |
| 항상 켜진 Linux 서버 또는 안정적인 실행 환경 | Webhook 수신기를 계속 실행해야 함 |
| 고정 HTTPS 주소 | GitHub가 이벤트를 전달할 주소 |
| Webhook Secret | GitHub 요청 서명 검증에 사용 |

GitHub Token, Slack Token, Webhook Secret은 Git·문서·AI 채팅에 넣지 않습니다. 서버의 비공개 환경 변수 또는 Secret 저장소에만 보관합니다.

## 권장 구조

```text
GitHub Issues / Issue comments
             │
             │  서명된 Webhook
             ▼
작은 Webhook Bridge 서비스
  ├─ 요청 서명 검증
  ├─ 중복 전달 제거
  ├─ Issue ↔ Slack 연결 정보 조회
  └─ Slack API로 부모글 또는 스레드 답변
             │
             ▼
      Slack Issue 전용 스레드
```

이 Bridge는 Hermes Gateway와 역할이 다릅니다.

- **Hermes Gateway:** Slack·Telegram의 대화 메시지를 Hermes에 전달
- **Webhook Bridge:** GitHub 이벤트를 검증하고 연결된 Slack 스레드에 반영

따라서 GitHub↔Slack 동기화를 쓰려면 Gateway 외에 작은 Bridge 서비스를 별도로 운영합니다.

## 꼭 저장할 연결 정보

Issue 번호만으로 Slack 스레드를 찾으면 안 됩니다. 새 Issue를 Slack에 처음 게시할 때 아래 정보를 영구 저장합니다.

```text
repository
issue_number
issue_url
slack_channel_id
slack_parent_thread_ts
posted 상태
처리한 GitHub delivery ID 목록
```

이 정보가 있어야 서비스 재시작, GitHub 재전송, 나중의 Slack 대화에서도 같은 스레드를 정확히 찾을 수 있습니다.

### 중복 방지 원칙

GitHub Webhook은 같은 이벤트를 다시 보낼 수 있습니다. Bridge는 `X-GitHub-Delivery` 같은 전달 ID를 기록해 이미 처리한 이벤트를 무시해야 합니다.

Slack 부모글은 외부 호출 전에 `posting` 상태를 먼저 기록하고, Slack이 성공 응답을 준 뒤에만 `posted`와 `thread_ts`를 저장합니다. 호출 결과가 불확실하면 재시도하기 전에 먼저 확인합니다. 그렇지 않으면 부모글이 중복될 수 있습니다.

## GitHub Webhook 등록

GitHub 저장소에서 다음으로 이동합니다.

```text
Settings → Webhooks → Add webhook
```

입력값은 다음 기준을 따릅니다.

| 항목 | 값 |
|---|---|
| Payload URL | Bridge의 고정 HTTPS 주소 |
| Content type | `application/json` |
| Secret | 서버에 보관한 Webhook Secret과 같은 값 |
| SSL verification | 활성화 |
| Events | `Issues`, `Issue comments` |

Bridge는 원본 요청 본문과 `X-Hub-Signature-256`을 사용해 HMAC-SHA256 서명을 검증해야 합니다. 서명이 없거나 일치하지 않는 요청은 처리하지 않고 거부합니다.

외부 공개 서버가 없다면 Cloudflare **Named Tunnel**처럼 고정 주소를 제공하는 방법을 쓸 수 있습니다. 임시 주소가 매번 바뀌는 Quick Tunnel은 운영용으로 적합하지 않습니다.

## Slack에 필요한 최소 권한

기본 [Slack 봇 연결 가이드](slack-bot-setup.md)의 권한에 아래만 추가하면 됩니다.

```text
chat:write       # 부모글과 스레드 답변
reactions:write  # 완료 시 ✅ 표시
```

scope를 추가했다면 Slack에서 **Reinstall to Workspace**를 실행합니다. Bot Token이 새로 발급되면 사용자가 Hermes/Bridge의 비공개 설정에 새 값을 직접 반영합니다.

연결 정보를 저장한다면 Slack 전체 채널 기록을 검색할 필요가 없으므로, 광범위한 `channels:history` 권한을 동기화 용도로 추가하지 않아도 됩니다.

## 상태를 반영하는 방법

Slack 부모글은 Issue의 고정 진입점입니다. 부모글 본문에 `상태: 진행 중`처럼 변하는 상태를 고정해 두지 않습니다. 나중에 완료되어도 오래된 상태 문구가 남기 때문입니다.

```text
부모글: Issue 제목 · 번호 · 링크
스레드 답글: 상태 변경, 댓글, 검수 안내
완료 시: 부모글 ✅ + 스레드 완료 안내
```

라벨·상태는 GitHub Issue가 기준입니다. Bridge는 GitHub의 확정된 변경을 Slack에 반영만 합니다. Webhook을 받아 라벨을 다시 수정하는 방식은 전달 지연과 재전송 때 반복 루프를 만들 수 있습니다.

## 설치 뒤 테스트 순서

1. 테스트용 Issue 하나를 만들고 Slack 부모글이 정확히 한 번 생성되는지 확인합니다.
2. 같은 Webhook delivery를 재전송해도 부모글·답글이 중복되지 않는지 확인합니다.
3. 테스트 Issue에 댓글을 달고 같은 Slack 스레드에만 전달되는지 확인합니다.
4. 테스트 Issue의 상태를 바꾸고 스레드에 한 번만 안내되는지 확인합니다.
5. 연결 정보가 없는 Issue 이벤트는 Slack에 아무 작업도 하지 않는지 확인합니다.
6. 완료 이벤트에서는 `✅`와 완료 안내가 모두 성공했을 때만 완료로 기록합니다.

## 운영 원칙

- 영향 있는 작업은 사람의 명시적 승인 뒤에만 수행합니다.
- Webhook 서비스는 읽기·알림·기록 연결부터 시작합니다.
- LLM은 댓글 요약이나 답변 초안에는 사용할 수 있지만, 중복 방지·서명 검증·상태 저장은 결정적인 코드로 처리합니다.
- 토큰·서명 값은 로그에 출력하지 않습니다.

## 공식 참고 자료

- [GitHub: Webhook 서명 검증](https://docs.github.com/en/webhooks/using-webhooks/validating-webhook-deliveries)
- [GitHub: Webhook 이벤트와 payload](https://docs.github.com/en/webhooks/webhook-events-and-payloads)
- [Slack 봇 연결 가이드](slack-bot-setup.md)
