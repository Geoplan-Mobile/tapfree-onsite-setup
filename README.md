# Tapfree 현장 설치 가이드

> **대상 독자**: 해외 고객사의 현장 설치 담당자.
> **목적**: Tapfree 솔루션 현장 설치에 필요한 **인프라 사전 세팅** 과 **Tapfree-edge 설정** 을 직접 수행할 수 있도록 안내한다.

---

## 본 문서의 범위

본 문서는 두 단계로 나뉜다.

1. **인프라 설정** — EdgePC 하드웨어/네트워크/시간/Geospace/PRM 준비
2. **Tapfree-edge 설정** — Tapfree-edge 솔루션 자체의 현장별 상세 설정

### 포함되지 않는 범위 (별도 영역)

| 항목 | 사유 |
|---|---|
| EdgePC 에의 Geospace / PRM / Tapfree-edge / Quuppa 소프트웨어 설치 | 사전 설치 완료 상태로 제공 |
| 앵커 박스 / 게이트 / 캘리브레이션 지점의 위치 좌표 측정 방법 | 서비스팀 교육 |
| Quuppa 상세 설정 (앵커 위치 등록, 측위 영역 설정 등) | 서비스팀 교육 |

본 문서는 **앵커 위치 좌표 · 게이트 위치 좌표 · 캘리브레이션 측정 지점 좌표가 모두 확정된 상태**에서 시작한다고 가정한다.

---

## 전체 흐름

```
─── 1. 인프라 설정 ───
[하드웨어 배선]
        ↓
[LTE 모뎀 설정]
        ↓
[EdgePC TimeZone 설정]
        ↓
[Geospace 앵커 설정]
        ↓
[PRM 게이트 / 영역 설정]
        ↓
[iPhone 캘리브레이션]
        ↓
─── 2. Tapfree-edge 설정 ───
[Tapfree-edge 설정 (zone_settings.json)]
```

---

## 단계별 문서

### 1. 인프라 설정

| # | 단계 | 문서 |
|---|---|---|
| 00 | 전제 조건 확인 | [00-prerequisites.md](./00-prerequisites.md) |
| 01 | 하드웨어 배선 | [01-hardware-wiring.md](./01-hardware-wiring.md) |
| 02 | LTE 모뎀 설정 | [02-lte-modem.md](./02-lte-modem.md) |
| 03 | EdgePC TimeZone 설정 | [03-edgepc-timezone.md](./03-edgepc-timezone.md) |
| 04 | Geospace 앵커 설정 | [04-geospace-anchors.md](./04-geospace-anchors.md) |
| 05 | PRM 게이트 / 영역 설정 | [05-prm-gates-areas.md](./05-prm-gates-areas.md) |
| 06 | iPhone 캘리브레이션 | [06-iphone-calibration.md](./06-iphone-calibration.md) |

### 2. Tapfree-edge 설정

| # | 단계 | 문서 |
|---|---|---|
| 07 | Tapfree-edge 설정 (zone_settings.json) | [07-tapfree-edge-config.md](./07-tapfree-edge-config.md) |

각 단계는 **순차적으로** 진행해야 한다. 이전 단계가 완료되지 않은 상태에서 다음 단계를 시작하면 검증이 불가능하다.

---

## 시스템 구성 요소 개요

| 구성 요소 | 역할 |
|---|---|
| **EdgePC (CM5)** | Tapfree-edge 서버를 구동하는 현장 컨트롤러. Geospace / PRM / Quuppa 도 함께 구동. |
| **LTE 모뎀** | EdgePC 의 외부망 연결. DDNS 를 통해 모바일 클라이언트가 접속. |
| **앵커 박스** | 박스 1기당 **Quuppa 앵커 1기 + UWB 앵커 1기** 가 함께 설치되어 동일 위치 좌표를 공유.  |
| **BLE 모듈** | Zone / Aisle 인식용 BLE 비콘. 모바일 앱이 스캔하여 zone 진입을 감지. |
| **Gate 모듈** | 결제/통과 게이트 (타 시스템). EdgePC 와 유선 연결. |

---

## 진행 전 준비물 체크리스트

- [ ] 기본 프로그램이 설치완료된 EdgePC(CM5)
- [ ] LTE 모뎀 — 현지 통신사 개통, DDNS / DHCP / DMZ / Wi-Fi 지원
- [ ] PoE 스위치 1대, BLE 모듈 2기, 앵커 박스 4기 (각 박스 = Quuppa + UWB)
- [ ] 앵커 위치 좌표 목록 
- [ ] 게이트 위치 좌표 목록 
- [ ] 캘리브레이션 측정 지점 좌표
