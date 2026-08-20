# 원격 Ubuntu 서버에 Hermes 설치하기

이 문서는 원격 Ubuntu 서버에 Hermes를 설치하고, 필요하면 메신저 봇으로 계속 실행할 때 사용합니다.

## 시작 전에 내가 준비할 것

1. 서버의 SSH 주소·계정·개인키
2. 서버에서 패키지를 설치할 수 있는 `sudo` 권한
3. 사용할 AI 모델 계정
4. 메신저를 연결한다면 Telegram 또는 Slack의 관리 권한

## 1. SSH 별칭 만들기

작업자 PC의 `~/.ssh/config`에 서버 별칭을 등록합니다.

```text
Host my-hermes
    HostName <서버 주소>
    User <서버 계정>
    IdentityFile ~/.ssh/<개인키 파일>
    IdentitiesOnly yes
```

접속이 비밀번호 없이 되는지 먼저 확인합니다.

```bash
ssh my-hermes 'whoami; hostname'
```

이 단계가 실패하면 Hermes 설치보다 SSH 설정을 먼저 해결합니다.

## 2. 서버를 먼저 점검하기

설치 전에 OS·저장 공간·기존 Hermes·systemd 상태를 확인합니다.

```bash
ssh my-hermes '
  cat /etc/os-release | head -3
  df -h /
  command -v git curl python3
  systemctl --user is-system-running
  loginctl show-user "$USER" -p Linger
'
```

기존 Hermes가 있다면 새로 덮어쓰지 말고 `hermes doctor`, `hermes gateway status`부터 확인합니다.

## 3. 설치와 모델 연결

서버에 접속해 설치합니다.

```bash
ssh -t my-hermes
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
source ~/.bashrc
hermes setup
```

모델 로그인·토큰 입력은 서버 화면에서 본인이 직접 합니다. AI에게 비밀번호나 토큰을 보내지 않습니다.

## 4. 메신저 봇으로 계속 실행하기

Telegram·Slack처럼 Gateway가 필요한 기능을 연결한 뒤 설치합니다.

```bash
hermes gateway setup
hermes gateway install
hermes gateway status
```

서버 재부팅 뒤에도 사용자 서비스가 유지되도록 확인합니다.

```bash
systemctl --user is-enabled hermes-gateway.service
systemctl --user is-active hermes-gateway.service
loginctl show-user "$USER" -p Linger
```

모두 정상이라면 `enabled`, `active`, `Linger=yes`가 표시됩니다.

## 5. 완료 기준

아래를 모두 실제로 확인해야 합니다.

- `hermes doctor`에 치명적인 오류가 없음
- `hermes`에서 모델이 응답함
- 연결한 메신저에서 테스트 메시지를 보내 응답을 받음
- Gateway가 `active` 상태임

## AI에게 맡길 때의 안전 원칙

AI는 서버 점검·설치·검증을 도울 수 있습니다. 다만 비밀값이 필요한 곳에서는 멈춰야 합니다.

```text
서버 실사와 설치는 진행해줘.
토큰·비밀번호·OAuth 로그인 입력이 필요하면 tmux 세션으로 넘기고, 내가 직접 입력할 때까지 기다려줘.
입력 후에는 비밀값이 화면이나 로그에 남지 않았는지 확인해줘.
```
