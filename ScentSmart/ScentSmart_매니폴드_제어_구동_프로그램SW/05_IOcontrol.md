
# IOControl.cpp / IOControl.h 상세 설명

## 1. 전체 역할

- 이 파일은 하드웨어 MCU에서 **입출력 핀 제어**를 담당합니다.
    
- 디지털 입력(버튼, 주소 스위치)과 출력(솔레노이드 밸브, LED, RS485 방향제어 등)을 초기화하고 제어하는 함수들이 구현됨.
    
- **향 분사기(솔레노이드 밸브), 세정밸브, 상태 표시 LED 등 하드웨어 장치 직접 제어가 핵심입니다.**
    

## 2. 주요 전역 변수

c

`char SolControlStatus[9] = { '0', '0', '0', '0', '0', '0', '0', '0', '0'};`

- **솔레노이드 밸브 9개 상태 저장 배열**
    
- 0: OFF, 1: ON을 문자로 표현하여 쉽게 상태 추적 가능하게 함.
    
- 각 인덱스가 특정 솔레노이드 밸브에 대응 (예: 인덱스 0이 `_D0_SOL0`)
    

## 3. `InitIO()` 함수

## 역할

- MCU의 각 입출력 핀을 **입력 또는 출력 모드로 설정**하고,
    
- 모든 액추에이터(솔레노이드 등 밸브들)는 초기화 시 모두 OFF 상태로 세팅합니다.
    

## 상세 동작

- **주소 핀(_DI_ADDR0 ~ _DI_ADDR3)**
    
    - `INPUT_PULLUP`으로 설정
        
    - 외부 스위치나 하드웨어 주소 설정용 입력핀, 풀업으로 기본 HIGH 유지
        
- **솔레노이드 핀(_D0_SOL0 ~ _D0_SOL8)**
    
    - `OUTPUT` 모드로 설정
        
    - 모두 `SOL_OFF` (OFF 상태)로 초기화 → 하드웨어 밸브 모두 닫힘
        
    - `SolControlStatus` 배열 0으로 초기화
        
- **기타 핀들**
    
    - `RS485` 통신 흐름 제어 핀 설정 (_DO_485_RE)
        
    - LED 상태 표시 핀 (_DO_LED_STS) → 초기 켜짐 또는 꺼짐 조절
        
    - Watchdog 신호 핀 (_DO_WDI) → 초기 HIGH 설정
        

## 4. `GetAddress()` 함수

## 역할

- 하드웨어 주소 설정 스위치 상태를 읽어,
    
- 4개의 주소 핀 `_DI_ADDR0 ~ _DI_ADDR3` 값을 0/1로 받아서
    
- **하드웨어 장치 주소를 0~15 사이 값으로 반환** (4bit)
    

## 동작 설명

- 각 주소핀은 `digitalRead()`를 사용해 읽고,
    
- ACTIVE LOW이므로 `!digitalRead()` (off 일 때 1)
    
- 비트 시프트하여 한 바이트에서 4개의 비트 위치에 맞게 합침
    
- 예:
    
    - `_DI_ADDR0` LOW → bit0 = 1
        
    - `_DI_ADDR1` HIGH → bit1 = 0
        
    - 반환값: (bit3bit2bit1bit0) 주소값 4비트 완성
        

## 5. `LiveSignal()` 함수

## 역할

- WatchDog 타이머 역할 수행 (장비가 정상 동작하는지 신호를 주기적으로 보내야 하는 장치에서 필수)
    
- 전송 제어 핀 `_DO_WDI` 상태를 반전(toggle)시켜서 생명신호를 보내며
    
- `WatchDogLive()` 함수 호출 (별도 구현)
    

## 6. `SolOn(int Pin)`, `SolOff(int Pin)` 함수들

## 역할

- 솔레노이드 밸브 ON/OFF 제어
    
- 입력으로 넘어온 핀 번호(`_D0_SOL0` 등)에 `digitalWrite`로 출력 HIGH/LOW 제어
    

## 상태 저장

- `switch` 문으로 각 핀별로 `SolControlStatus[]` 배열의 해당 인덱스 위치를 '1'(ON) 또는 '0'(OFF)로 업데이트
    

## 역할 한눈에

|함수|동작|
|---|---|
|SolOn(Pin)|지정 핀 신호 HIGH 출력 + 상태 '1' 기록|
|SolOff(Pin)|지정 핀 신호 LOW 출력 + 상태 '0' 기록|

## 7. 상태 LED 제어 함수

- `StatusLEDOn()`: LED 핀 `_DO_LED_STS`에 HIGH 출력 (LED 켜기)
    
- `StatusLEDOff()`: LED 핀 `_DO_LED_STS`에 LOW 출력 (LED 끄기)
    

## 8. RS485 통신 방향 제어

RS485 통신 트랜시버 모드 전환용 핀 제어

- `RS485_Rx_Enable()` :  
    RS485 수신 모드 (RE 핀 LOW)
    
- `RS485_Tx_Enable()` :  
    RS485 송신 모드 (RE 핀 HIGH)
    

## 9. 솔레노이드 밸브 일괄 초기화

- `ResetSol()`: 모든 솔레노이드 밸브 OFF 시킴 (0번부터 8번까지)
    

## 10. 향 분사용 솔레노이드 세팅 함수들

- `ReadyScentSol(int ScentPort)`
    
    |입력 `ScentPort` 값|동작|
    |---|---|
    |1|솔레노이드 1 OFF, 3, 5, 7 ON 세팅 (비교적 특정 향 준비를 위한 조합)|
    |2|1 ON, 3 OFF, 5 ON, 7 ON|
    |3|1 ON, 3 ON, 5 OFF, 7 ON|
    |4|1 ON, 3 ON, 5 ON, 7 OFF|
    
- `ReadyClearSol(int ClearPort)`
    
    |입력 `ClearPort`|동작|
    |---|---|
    |1|2 ON, 4 OFF, 6 OFF, 8 OFF|
    |2|2 OFF, 4 ON, 6 OFF, 8 OFF|
    |3|2 OFF, 4 OFF, 6 ON, 8 OFF|
    |4|2 OFF, 4 OFF, 6 OFF, 8 ON|
    
- 이 둘은 향 분사 및 세정 시 밸브를 포트별로 다르게 열기 위한 조합 제어 함수입니다.
    

## 11. 개별 솔레노이드 제어 함수

- `Sol_0_On() ~ Sol_8_On()`, `Sol_0_Off() ~ Sol_8_Off()`
    
- 각 함수는 단순히 `SolOn(_D0_SOLX)` 또는 `SolOff(_D0_SOLX)`을 호출, 단일 솔레노이드 개별 켜기/끄기 용
    

## 12. 헤더파일 IOControl.h 소개

- 글로벌 변수, 함수 프로토타입이 정의됨
    
- 전역 `SolControlStatus` 배열 및 핀 번호 배열 `DigitalWritePin[]` 선언
    
- IO 제어 관련 함수 선언
    

# 정리 및 이해 포인트

|기능|사용 예시 / 설명|
|---|---|
|디지털 핀 초기화|`InitIO()`로 모든 입출력핀 설정 및 초기 상태 지정|
|주소 읽기|`GetAddress()`로 하드웨어 장치별 고유 주소 결정|
|밸브 제어|`SolOn()`, `SolOff()`로 특정 솔레노이드 켜고 끔|
|상태 저장|`SolControlStatus[]` 배열에서 ON/OFF 상태 문자로 추적|
|RS485 송수신 모드 전환|`RS485_Tx_Enable()`, `RS485_Rx_Enable()`로 통신 방향 스위칭|
|상태 LED & 신호 유지|`StatusLEDOn()`, `StatusLEDOff()`, `LiveSignal()` 등|
|전체 밸브 초기화|`ResetSol()`으로 모든 밸브 끄기|
|향/세정 준비 밸브 조합|`ReadyScentSol()`, `ReadyClearSol()`로 포트별 밸브 조합 제어|
|개별 밸브 ON/OFF|각 솔레노이드별 함수로 빠르고 명확하게 제어 가능|
