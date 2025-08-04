# 🌬️ 디지털 향기 제어 통신 방식 비교 (MODBUS vs Legacy)

  

이 문서는 `ScentSmart.py` 기반 디지털 향기 제어 시스템에서 사용되는 두 가지 통신 방식의 구조를 비교 설명합니다.

  

---

  

## 🧭 전체 통신 흐름 비교

  

### ✅ MODBUS 방식

  

- 국제 표준 통신 규격 (산업용)

- `dsComm.py`에서 MODBUS 패킷 구성 (Function code, CRC 등 포함)

- 펌웨어에서는 CRC16 검증 → 명령 처리

- **다수 장치(Slave), 확장성, 무결성에 강함**

  

```mermaid
flowchart TD
    A[ScentSmart.py - 기능 호출] --> B[dsComm.py - MODBUS 패킷 조립]
    B --> C[Serial.write() - 송신]
    C --> D[MODBUS 펌웨어 - CRC 검증 및 파싱]
    D --> E[향기 제어 - ControlPumpNF.c]
```


---

  

### ✅ Legacy 방식 (Custom Serial)

  

- 자체 정의된 문자 기반 프로토콜

- `dsComm.py`에서 `[STX][Addr][Command][Data...][XOR][ETX]` 형식 조립

- 펌웨어는 패킷 파싱 및 명령 수행

- **빠르고 단순하지만 외부 연동/표준성은 부족**

  

```mermaid
flowchart TD
    A[ScentSmart.py<br>(기능 호출)] --> B[dsComm.py<br>(Legacy 패킷 구성 - STX~ETX)]
    B --> C[Serial.write()<br>직렬 송신]
    C --> D[펌웨어(CustomSerial)<br>명령 문자 기반 파싱]
    D --> E[향기 제어 실행<br>(ScentControl.c 등)]
```


  

---

  

## 🔍 핵심 비교 요약

  

| 항목 | MODBUS 스타일 | Legacy 스타일 |

|------|----------------|----------------|

| 프로토콜 | 국제 표준 (MODBUS RTU) | 커스텀 ASCII 기반 |

| 체크섬 | CRC16 | XOR 2바이트 |

| 다채널 제어 | O (레지스터 기반) | O (명령 기반) |

| 다수 장치 제어 | O (Slave 주소) | X |

| 외부 호환성 | ✅ 높음 (PLC, HMI 등) | ❌ 없음 |

| 개발 난이도 | 비교적 높음 | 낮음 |

| 디버깅 툴 | MODSCAN, modbus-py 등 | 자체 시리얼 로그 |

  

---

  

## 💡 정리

  

- 향기 채널 1, 3, 8번을 조합하는 동작은 **"1개 장치 안의 다채널 제어"**입니다.

- 현재 `ScentSmart.py`는 **기능 제어만 담당**하고,  

  실제 패킷 형식(MODBUS 또는 Legacy)은 **`dsComm.py`가 결정**합니다.

- 소프트웨어 코드는 바꾸지 않고, **dsComm을 교체하여 두 펌웨어 모두 호환 가능**한 구조입니다.