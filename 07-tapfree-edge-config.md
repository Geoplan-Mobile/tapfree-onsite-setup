# 07. Tapfree-edge 설정 (zone_settings.json)

← [06. iPhone 캘리브레이션](./06-iphone-calibration.md)

Tapfree-edge 솔루션의 현장별 상세 설정은 **`zone_settings.json` 파일 편집** 으로 끝난다. 본 단계에서는 그 파일을 열어 현장 정보를 채워 넣고 서비스를 재시작한다.

> **선행 조건**
> - [04. Geospace 앵커 설정](./04-geospace-anchors.md) 완료 (앵커 MAC 확정)
> - [05. PRM 게이트 / 영역 설정](./05-prm-gates-areas.md) 완료 (게이트 ID 및 통로 ID 확정)
> - [06. iPhone 캘리브레이션](./06-iphone-calibration.md) 완료
> - [02. LTE 모뎀 설정](./02-lte-modem.md#ssh-접속-확인) 의 SSH 접속이 가능한 상태

---

## 사전 준비 정보

`zone_settings.json` 편집 전에 다음 정보가 모두 준비되어 있어야 한다.

- [ ] **Quuppa 앵커 MAC 4개** (12자리 hex, 콜론 없음)
- [ ] **각 통로 (aisle) 정보** — `id` (PRM aisleName 과 매칭), `gateId` (EquipId. Gate 모듈이 사용하는 ID)
- [ ] **각 통로의 BLE 모듈 MAC** — free 측 / paid 측 각각
- [ ] **UWB 앵커 MAC 4개** (4자리 hex) — Geospace 등록값과 정확히 일치해야 함
- [ ] **UWB 앵커 offsetNs 4개** — 현장 측정값. 마스터 앵커는 `0.0`

---

## 1. EdgePC SSH 접속

```bash
ssh root@{host}
# 비밀번호: wldhvmffos#123
```

---

## 2. zone_settings.json 편집

파일 위치:
```
/home/geoplan/geospace/tapfree-edge/zone_settings.json
```

```bash
cd /home/geoplan/geospace/tapfree-edge
sudo nano zone_settings.json
```

---

## 3. zone_settings.json 설정 항목

### 3.1 전체 예시

```json
{
    "zoneCode": "01",
    "zoneIp": "192.168.0.30",
    "useZoneIp": false,
    "quuppa_no": {
        "a4da22ef60e3": "1",
        "a4da22ef5d27": "2",
        "a4da22ef60e2": "3",
        "a4da22ef60bb": "4"
    },
    "aisles": [
        {
            "id": "01",
            "gateId": "0001",
            "freeBle": {
                "mac": "00:60:37:A8:1B:82",
                "boardRssi": [-56, -20],
                "exitRssi":  [-64, -20]
            },
            "paidBle": {
                "mac": "00:60:37:80:C3:76",
                "boardRssi": [-64, -20],
                "exitRssi":  [-58, -20]
            }
        }
    ],
    "anchorInfo": {
        "sessionId": 4444,
        "offsets": [
            { "mac": "5555", "offsetNs":  0.0 },
            { "mac": "6666", "offsetNs": 12.3 },
            { "mac": "7777", "offsetNs": -1.3 },
            { "mac": "8888", "offsetNs":  4.7 }
        ]
    }
}
```

### 3.2 Zone

- **zoneCode** : 기본값 `"01"`. 같은 역사 내 zone 이 여러 개일 경우 구분을 위해 `02`, `03` 등으로 지정.
- **zoneIp** : 기본값 `"192.168.0.30"`. `useZoneIp=true` 일 때만 의미가 있다.
    - 모바일이 EdgePC 에 **직접 IP 로 접속**하는 시나리오에서 사용하는 주소.
    - 사용 환경에 따라 값이 달라진다:
        - **내부망 접속 (디버깅)** — EdgePC 의 내부 IP (예: `192.168.0.30`)
        - **외부망 접속이지만 DDNS 없이 LTE 모뎀 고정 IP 사용** — LTE 모뎀의 **고정 공인 IP**
- **useZoneIp** : 기본값 `false`.
    - `false` — AN200 Advertising 데이터에 역번호와 `zoneCode` 가 세팅. 모바일이 이 값으로 DDNS 도메인 URL 을 작성해 WebSocket 접속한다. **표준 운영 모드.**
    - `true` — AN200 Advertising 데이터에 위에서 설정한 `zoneIp` 가 세팅. 모바일이 그 IP 로 WebSocket 접속.
        - 사용 케이스 ①: **내부망 디버깅** — EdgePC 의 내부 IP 로 접속.
        - 사용 케이스 ②: **DDNS 미사용 + LTE 모뎀 고정 공인 IP** — `zoneIp` 에 모뎀의 고정 공인 IP 를 넣고 `true` 로 설정.

### 3.3 Quuppa

- **quuppa_no** : Quuppa 앵커 MAC → 번호(1~4) 매핑. **필수 필드, 정확히 4개 항목이어야 하며 값은 `"1"`, `"2"`, `"3"`, `"4"` 모두 포함해야 한다** (누락 또는 4 외의 값이 있으면 부팅 실패).
    - **key** : Quuppa 앵커의 MAC (12자리 hex, 콜론 없음, 대소문자 무관 — 내부에서 대문자로 정규화).
    - **value** : 그 앵커의 식별 번호 문자열 (`"1"` ~ `"4"`).
    - **용도** : 앵커에 문제가 발생했을 때 **게이트 측에 어느 앵커인지 알려주기 위한 식별자**. 게이트는 MAC 이 아닌 1~4 번호로 앵커를 구분하므로, 본 매핑으로 MAC ↔ 번호를 미리 정해 둔다.
    - 서버 내부 매핑 용도이며 모바일에는 노출되지 않는다.

### 3.4 Aisles

- **aisles** : 게이트 통로 정보의 리스트. 현장 통로 개수만큼 아래 객체가 존재.
    - **id** : 통로 ID (모바일에서 사용). geoplan 이 부여하는 통로 ID 이며 PRM 의 `aisleName` 과 매칭되어야 한다.
    - **gateId** (EquipId) : 실제 게이트 접속 시 사용되는 ID. 에스트래픽 측 Gate 모듈이 전달하는 ID 이며 외부 시스템에서는 **EquipId** 라는 명칭으로도 통용된다. 모바일에서는 사용되지 않으며 edge 가 `id` 와 `gateId` 로 모바일의 인식 통로와 게이트를 인터페이싱한다.
    - **freeBle** : free 측에 설치된 AN200 의 MAC 과 board / exit 판단 RSSI 범위.
        - 기본값 : `"boardRssi": [-56, -20]`, `"exitRssi": [-64, -20]`
    - **paidBle** : paid 측에 설치된 AN200 의 MAC 과 board / exit 판단 RSSI 범위.
        - 기본값 : `"boardRssi": [-64, -20]`, `"exitRssi": [-58, -20]`

### 3.5 AnchorInfo (DL-TDoA / UWB 측위)

- **anchorInfo** : DL-TDoA 측위에 사용되는 UWB 세션 설정.
    - **sessionId** : UWB ranging 세션 식별자 (정수). 현장 변경 사유 없으면 기본값 유지.
    - **offsets** : 앵커별 타이밍 오프셋 정보 배열.
        - **mac** : 앵커 MAC (4자리 hex, 대소문자 무관 — 내부에서 대문자로 정규화).
        - **offsetNs** : 마스터 앵커 대비 타이밍 오프셋(나노초 단위). **마스터 앵커는 `0.0` 으로 명시 기재 필요** (생략 불가).

> **검증 규칙** : `offsets` 의 MAC 집합은 **Geospace 에 등록된 앵커 MAC 집합과 정확히 일치**해야 한다 (양쪽 어디든 한 개라도 누락/추가되면 부팅 실패). 마스터 앵커도 반드시 `offsetNs=0.0` 로 명시되어 있어야 한다.

> 이 섹션은 부팅 시 PRM 의 area_info, Geospace 의 앵커 목록과 함께 로드되어 검증된다 — PRM / Geospace 가 응답하지 않거나 데이터가 불일치하면 tapfree-edge 가 시작되지 않는다.

편집 완료 후 저장: `Ctrl+O` → Enter → `Ctrl+X`.

---

## 4. 서비스 재시작

tapfree-edge 폴더에 재시작 스크립트가 준비되어 있다.

```bash
cd /home/geoplan/geospace/tapfree-edge
./restart.sh
```

상태 확인:
```bash
sudo systemctl status tapfree-edge
```

`Active: active (running)` 가 보이면 정상 기동.

> 같은 폴더에 `start.sh` / `stop.sh` 도 함께 있다 — 필요 시 시작/중지만 따로 수행 가능.

---

## 5. 설정 적용 확인

부팅 시 `zone_settings.json` 파싱 + Geospace / PRM 동기 검증이 수행되므로, **실패하면 서비스가 시작되지 않는다.**

로그 파일 위치:
```
/home/geoplan/geospace/tapfree-edge/logs/processor-tpe.log
```

다음 메시지가 보이면 정상:
- `zone settings initialized, zoneCode=01`
- `Prm Inout Area loaded : areaCount {N}`
- `Geospace anchors loaded : count {N}, macs: [...]`

자주 발생하는 오류:

| 메시지 | 원인 |
|---|---|
| `quuppa_no is null` | `quuppa_no` 블록 누락 |
| `quuppa_no must have 4 elements and values 1, 2, 3, 4` | 4개 미만이거나 1~4 외의 값 사용 |
| `anchor mac mismatch between Geospace and zone_settings.json` | Geospace 등록 MAC 과 `offsets` MAC 불일치. **마스터 앵커도 명시 필요** |
| `zoneCode or zoneIp is null` | `zoneCode` 또는 `zoneIp` 필드 누락 |

> 위 오류가 발생하면 **tapfree-edge 는 정상적으로 시작하지 못하고 프로세스가 종료된다.** 이후 systemd 가 일정 시간 단위로 **계속 재시작을 시도**하므로 로그에 같은 오류가 반복 기록될 수 있다. `zone_settings.json` 을 정정한 뒤 다시 시작해야 정상 기동된다.

---

## 완료 후 확인

- [ ] `zone_settings.json` 의 모든 블록이 현장값으로 작성됨
- [ ] `sudo systemctl status tapfree-edge` 가 `active (running)` 상태
- [ ] 로그에 `zone settings initialized` 메시지가 보임

---

이로써 Tapfree 솔루션 현장 설치가 완료되었다.
