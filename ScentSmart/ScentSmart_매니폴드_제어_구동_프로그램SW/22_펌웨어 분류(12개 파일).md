  

# 💡 디지털 향 분사 장치 제어 시스템 (AVR 펌웨어 분석)

  

이 시스템은 **PC 또는 상위 컨트롤러(Host)**로부터 **RS485 시리얼 명령**을 받아,

향기 분사/세정/중지/상태조회/밸브제어 등을 수행하는 **펌웨어 기반 디바이스**입니다.

  

---

  

## 📦 구성 파일 개요

  

| 파일명              | 역할 설명 |

|---------------------|-----------|

| `HostRun.*`         | 시리얼 명령 수신 → 파싱 → 실행 |

| `ScentControl.*`    | 향기/세정 동작 처리 로직 |

| `IOControl.*`       | 입출력 핀 제어 (SOL, LED 등) |

| `nfserial.*`        | 시리얼 클래스 구현 (UART 직접 제어) |

| `TimerOne.*`        | Timer1 기반 인터럽트 / PWM 제어 |

| `IODefine.h`        | IO 핀번호 및 상수 정의 |

| `known_16bit_timers.h` | 보드별 타이머 핀 정의 |

  

---

  

## 🔄 전체 동작 흐름

  

```text

[PC 명령 전송]

    ↓

Serial 수신 (NFSerial)

    ↓

STX~ETX 패킷 완성 (HostRun)

    ↓

Command 파싱 → 실행 함수 매핑

    ↓

ScentControl/IOControl 등 호출

    ↓

작동 결과 응답 패킷 전송

```

  

---

  

## 🧩 핵심 구조 분석

  

### 1. nfserial (시리얼 클래스)

  

- `SerialHost.begin(19200)` 로 UART0 시리얼 통신 시작

- 내부 수신 버퍼 / 송신 버퍼 사용

- ISR (USART0_RX_vect, UDRIE_vect) 통해 직접 동작

- `read`, `write`, `available` 등의 메서드 제공

  

### 2. HostRun (명령 수신 및 파싱)

  

- 패킷 구조: `STX + Addr[2] + Cmd[1] + Data[n] + Checksum[2] + ETX`

- 명령 목록:

  

| Cmd | 기능                 | 데이터 구조 |

|-----|----------------------|-------------|

| C   | 통신 확인            | 없음        |

| R   | 향기 분사            | Delay+번호+Period |

| L   | 세정                 | Delay+번호+Period |

| S   | 정지                 | Delay+번호+Period |

| Z   | 밸브 상태 브로드캐스트 | 17 bytes   |

| z   | 밸브 상태 설정 (단일) | 9 bytes    |

| v   | 밸브 상태 요청        | 없음        |

  

- 체크섬 방식: **XOR → 문자화 → 마지막 2자리 비교**

  

### 3. ScentControl (향기 동작)

  

- 주요 상태 변수: `RunScentLoopFlag`, `RunScentLoopIndex`, 등

- 향 분사: `DoScentBackLoop()` → `RunScentLoop()` 호출 루프

- 세정: `DoClearBackLoop()` → `RunClearLoop()`

- 정지: `DoStopBackLoop()`

  

### 4. IOControl (핀 제어)

  

- `SolOn(Pin)`, `SolOff(Pin)` 으로 직접 디지털 출력 제어

- `ReadyScentSol(ScentNo)` → 향기용 솔레노이드 조합 ON

- `ResetSol()` → 모든 솔레노이드 OFF

- `RS485_Tx_Enable()` / `Rx_Enable()` → 통신 방향 제어

  

---

  

## 🧾 IODefine 및 Timer

  

- **IODefine.h**: `_D0_SOL0`, `_DO_LED_STS` 등 디바이스 IO 매핑 정의

- **TimerOne**: 인터럽트 & PWM 기능 구현, 일부 기능에서 사용 가능성 있음

- **known_16bit_timers.h**: 보드별 Timer 핀 번호 정의

  

---

  

## ✅ 요약

  

- 이 펌웨어는 **모듈별로 분리되어 가독성과 유지보수에 강함**

- 모든 시리얼 명령은 **HostRun → ScentControl/IOControl** 경로로 처리됨

- 시리얼 버퍼, 체크섬, 인터럽트, IO 핀 모두 직접 제어하는 **로우레벨 AVR 제어 방식**

  

---

  

## 📌 참고

  

- 각 명령별 상세 동작, 구조체 정의, 테스트 시나리오 필요 시 따로 정리 가능

- Obsidian에서 `[[]]` 링크나 태그 구조로 이어서 확장 정리할 수 있음