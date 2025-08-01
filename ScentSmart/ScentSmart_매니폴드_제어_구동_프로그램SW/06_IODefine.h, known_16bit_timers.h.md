아래는 주신 `IODefine.h`와 `known_16bit_timers` 헤더 파일에 대한 **역할, 정의 내용, 동작 목적**을 아주 쉽고 상세하게 풀어 설명한 내용입니다.  
임베디드 하드웨어 펌웨어 개발, 특히 MCU 핀 할당 및 타이머 핀 정의를 이해할 때 꼭 알아야 하는 기본 구성입니다.

# 1. IODefine.h 파일 설명

`IODefine.h`는 **MCU(마이크로컨트롤러)를 구성하는 각 입출력 핀(포트번호)을 의미 있는 이름으로 정의해 둔 헤더파일**입니다.  
이를 통해 코드는 추상화되고, 하드웨어 구조 변경 시 이 파일만 수정하면 됩니다.

## 1-1. 주요 매크로 정의 의미

## (1) 입력 핀 주소 정의 (_DI_ADDRx)

c

`#define _DI_ADDR0 PIN_PE0 #define _DI_ADDR1 PIN_PE1 #define _DI_ADDR2 PIN_PE2 #define _DI_ADDR3 PIN_PE3`

- MCU 포트 E의 0~3번 핀(PE0~PE3)을 각각 4비트 하드웨어 주소 입력 핀으로 지정
    
- 보통 하드웨어 장치 주소 설정용 DIP 스위치 등으로 쓰임
    
- `PIN_PE0` 등은 MCU별 핀 번호(Atmel AVR 계열 등 내부 정의)
    

## (2) 출력용 솔레노이드 핀 (_D0_SOLx)

c

`#define _D0_SOL0  PIN_PB0 #define _D0_SOL1  PIN_PB1 #define _D0_SOL2  PIN_PB2 #define _D0_SOL3  PIN_PC0 #define _D0_SOL4  PIN_PC1 #define _D0_SOL5  PIN_PC2 #define _D0_SOL6  PIN_PC3 #define _D0_SOL7  PIN_PC4 #define _D0_SOL8  PIN_PC5`

- 솔레노이드 밸브(향 분사용 밸브) 9채널 각각에 대응하는 MCU 핀 설정
    
- 포트 B0, B1, B2, C0~C5 핀에 연결됨
    
- `PIN_PBx`, `PIN_PCx`는 MCU 핀 번호 매크로
    

## (3) 기타 출력을 위한 핀

c

`#define _DO_WDI       PIN_PD7  // 워치독 신호용 출력 #define _DO_485_RE    PIN_PD2  // RS485 통신 송수신 방향 제어 핀 (HIGH 송신) #define _DO_LED_STS   PIN_PB5  // 상태 표시 LED 핀`

- **워치독 신호(WDI)**: MCU 정기 작동 신호용 핀
    
- **RS485 송수신 제어**: 통신 모드 전환을 위한 핀 (HIGH일 때 송신 모드)
    
- **상태 LED**: 장비 상태 표시용 LED 제어 핀
    

## 1-2. 핀 상태 매크로

c

`#define SOL_OFF LOW #define SOL_ON  HIGH #define LED_OFF     HIGH #define LED_ON      LOW`

- 솔레노이드 밸브 출력 신호 논리값 정의 (`ON`은 `HIGH`, `OFF`는 `LOW`)
    
- LED 신호는 반대 (On일 때 LOW 전압, Off일 때 HIGH 전압)
    
    - 이는 회로 설계상 LED가 `액티브 로우` 방식일 경우임
        

## 1-3. 코드를 왜 이렇게 작성하나?

- **직관적 이름 부여** → 코드 유지보수가 쉬움
    
- MCU 핀 번호가 바뀌면 이 파일만 수정으로 대응 가능
    
- 하드웨어 상세를 숨기고 개발 시 추상화된 핀 이름 사용
    

예)

c

`digitalWrite(_D0_SOL1, SOL_ON);`

→ 물리적 `PIN_PB1` 핀을 켜는 명령임

# 2. known_16bit_timers 파일 설명

이 파일은 여러가지 **AVR 및 Teensy 등 다양한 MCU 종류별 16비트 타이머 관련 핀 지정 매크로**를 모아둔 헤더파일입니다.  
주로 PWM 신호, 입력 캡처, 타이머 채널용 핀을 정의해 하드웨어별 차이를 통일하는 역할을 합니다.

## 2-1. 왜 필요한가?

- AVR, Teensy, Arduino 등 MCU마다 16비트 타이머 하드웨어의 PWM 출력이나 캡처 입력 핀이 다 다름
    
- 타이머 관련 기능을 쓰기 위해선 해당 핀 번호를 알아야 하며, 보통 특정 MCU 데이터시트와 맞추어야 함
    
- 이를 위한 표준화된 이름 저장소 역할
    

## 2-2. 구조 및 이해

- 여러 MCU별로 조건 컴파일(if defined...)로 나누어 각 MCU의 타이머 핀을 정의
    
- 예시:
    

c

`#if defined(__AVR_ATmega328P__)  // Arduino Uno 보드 칩   #define TIMER1_A_PIN   9    // Arduino Digital Pin 9 = OC1A   #define TIMER1_B_PIN   10   // Arduino Digital Pin 10 = OC1B   #define TIMER1_ICP_PIN 8    // Input Capture Pin   #define TIMER1_CLK_PIN 5    // External Clock Pin #endif`

- `TIMERx_A_PIN` 은 타이머x 채널A 출력 핀번호
    
- `TIMERx_B_PIN` 은 채널B
    
- `TIMERx_ICP_PIN` 은 입력 캡처 및 타이머 입력용 핀
    
- `TIMERx_CLK_PIN` 은 외부 클록 입력 핀
    

## 2-3. 지원 MCU 목록 예시

- Wiring-S (ATmega644P)
    
- Teensy 2.0 (ATmega32U4)
    
- Teensy++ 2.0 (AT90USB1286)
    
- Teensy 3.x 시리즈 (MK20DX128, MK20DX256, MK64FX512, MK66FX1M0, MKL26Z64, IMXRT1062)
    
- Arduino Mega (ATmega1280, ATmega2560)
    
- Arduino Leonardo / Yun (ATmega32U4)
    
- Arduino Uno, Duemilanove, LilyPad (ATmega168, ATmega328P, ATmega8)
    
- Minicore 또는 Sanguino 등 (다양한 AVR 기반 MCU)
    

이렇게 각 보드별 타이머 및 핀을 일괄 관리하여, 펌웨어에서 조건부 컴파일할 때 쉽게 사용

## 2-4. MCU 핀 번호와 Arduino 디지털 핀 넘버 차이

- MCU 핀명(예: PORTB0, PE0 등)은 하드웨어 핀 번호이고,
    
- Arduino 디지털 핀 번호는 개발 보드상의 대표 번호
    

예시: ATmega328P의 PIN 9은 MCU PORTB1에 할당됨

# 3. 요약 & 사용 의미

|파일명|역할|주요 활용|
|---|---|---|
|`IODefine.h`|MCU 각 핀에 대한 상수 이름 정의|하드웨어 제어(입출력) 시 코드 내 핀 번호 역할|
|`known_16bit_timers`|다양한 MCU별 16비트 타이머 관련 핀 정의|MCU 타이머 채널 사용 시 어떤 핀으로 동작하는지 참조하고 설정|

# 4. 쉽게 비유

- `IODefine.h`는 **하드웨어 핀에 붙이는 “별명표”**입니다
    
- `known_16bit_timers`는 **여러 MCU별로 “타이머 단골자리 번호“를 정리한 안내서**라고 보면 됩니다.
    

필요하면 예시 코드(핀 제어, PWM 설정 등)나 각 MCU별 핀 매핑 표, 그리고 타이머 활용법도 쉽게 설명해 드릴 수 있으니 편히 요청해 주세요!