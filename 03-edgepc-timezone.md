# 03. EdgePC TimeZone 설정

← [02. LTE 모뎀 설정](./02-lte-modem.md)

EdgePC 의 시스템 타임존을 **설치 지역에 맞게** 변경한다.

본 설정의 목적은 **시스템 간 시각 동기화** — Tapfree 솔루션을 구성하는 각 시스템(EdgePC, Edge Server, 모바일, 게이트 등)이 동일 현지 시각 기준으로 동작하도록 맞추기 위함이다. 타임존이 일치하지 않으면 로그 시각, 이벤트 타임스탬프 등이 어긋나 장애 추적이 어려워진다.

> **기본값은 `KST` (Asia/Seoul)** 로 출하된다. 한국 외 지역에 설치하는 경우 반드시 본 단계에서 현지 타임존으로 변경해야 한다.

---

## 1. EdgePC SSH 접속

[02. LTE 모뎀 설정](./02-lte-modem.md#ssh-접속-확인) 의 점검 절차로 외부망에서 SSH 접속이 가능한 상태여야 한다.

```bash
ssh root@{host}
# 예) ssh root@tapfreeE0801.cns-link.net
```

비밀번호 : `wldhvmffos#123`

---

## 2. 현재 타임존 확인

```bash
timedatectl
```

`Time zone:` 줄에서 현재 설정된 값을 확인.

---

## 3. 타임존 변경

```bash
sudo timedatectl set-timezone {timezone}
```

`{timezone}` 자리에 설치 지역에 해당하는 값을 채워 넣는다.

### 주요 사용 타임존

| 지역 | timezone 값 |
|---|---|
| 한국 (Seoul) | `Asia/Seoul` |
| 미국 — 샌프란시스코 (Pacific Time) | `America/Los_Angeles` |
| 미국 — 워싱턴 (Eastern Time) | `America/New_York` |

### 예시

```bash
# 샌프란시스코 설치 시
sudo timedatectl set-timezone America/Los_Angeles
```

---

## 4. 시스템 재부팅

**타임존 변경 후 EdgePC 를 반드시 재부팅** 해야 변경 사항이 모든 구동 중인 서비스(Tapfree-edge, Geospace, PRM, Quuppa 등)에 반영된다.

```bash
sudo reboot
```

재부팅 완료 후 다시 SSH 로 접속한다.

---

## 5. 변경 결과 확인

```bash
timedatectl
```

- `Time zone:` 줄이 지정한 값으로 바뀌었는지 확인
- `Local time:` 의 현재 시각이 현지 시각과 일치하는지 확인

---

→ [04. Geospace 앵커 설정](./04-geospace-anchors.md)
