---
aliases:
  - 04_HostRun cpp/header
---


# 1. 펌웨어가 하는 일 - 전체 개념 정리

이 펌웨어는 PC(상위 소프트웨어)와 **시리얼 통신**으로 데이터를 주고받으며,  
받은 명령에 따라서 하드웨어 향 분사기, 세정기, 밸브 등을 제어하는 역할을 합니다.

# 2. 시리얼 통신 데이터의 기본 구조

- 데이터는 **문자 기반 ASCII 패킷** 형태로 주고받아요.
    
- 패킷 구조:
    

`STX(0x02)` 시작 + `주소(2글자)` + `명령 문자(1개)` + `데이터 (n 글자)` + `체크섬(2글자)` + `ETX(0x03)` 끝

예를 들면 이렇게요:  
`0x02 '0' '1' 'R' <12글자 데이터> <2글자 체크섬> 0x03`

# 3. 데이터 수신 흐름(중심 함수: `SerialHostEvent`)

- 시리얼 포트에 데이터가 들어오면 한 바이트씩 읽어요 (`SerialHost.read()`)
    
- 만약 이 바이트가 `STX(0x02)`라면:  
    → 수신 버퍼 초기화하고, 데이터를 새로 받는 신호로 저장
    
- 이후 들어오는 모든 데이터를 `SerialHostRxBuff` 버퍼에 차례대로 담고
    
- `ETX(0x03)`가 오면 패킷 수신 완료 플래그(`Host_CommandFlag`)를 세웁니다.
    
- 버퍼가 너무 커지면 자동 리셋하여 오버플로 방지
    

# 4. 수신 완료 후 처리 흐름(`RunSerialHostDataCheck`)

- `Host_CommandFlag`가 1이면 호출되어 처리 시작
    
- 수신 버퍼를 복사해서 명령 전용 버퍼에 저장
    
- 수신된 명령 문자(`SerialHostRxCommandBuff`)가 미리 정의된 명령 리스트에 있는지 검사(확인)
    
- 유효한 명령이면 그 명령의 데이터 크기만큼 파라미터를 따로 복사
    
- 명령 인덱스와 활성화 플래그를 설정하여 다음 `RunSerialHostCommand()`에서 처리 준비
    

# 5. 명령 처리 함수 `RunSerialHostCommand()`

- 수신 패킷 형식 체크 (STX 여부, 길이, 체크섬 검증)
    
- 자신의 주소와 맞는지 비교(다른 장치 명령 무시)
    
- 명령 문자에 따라 아래 함수를 호출
    

|명령 문자|의미|처리 함수|
|---|---|---|
|'C'|통신 확인|`RunSerialHostCommandConfirm()`|
|'R'|향 분사 명령|`RunSerialHostCommandRunScent()`|
|'L'|세정 명령|`RunSerialHostCommandRunClear()`|
|'S'|정지 명령|`RunSerialHostCommandRunStop()`|
|'G'|상태 요청|`RunSerialHostCommandGetStatus()`|
|'V'|버전 요청|`RunSerialHostCommandGetVersion()`|
|'Z','z'|솔레노이드 밸브 설정|`RunSerialHostCommandSetSolZeroValveBroadcast()`, `RunSerialHostCommandSetSolZeroValve()`|
|'v'|밸브 상태 조회|`RunSerialHostCommandGetValveStatus()`|

# 6. 명령 별 주요 처리 함수 동작 예시

## Run Scent 명령 (향 분사)

// 명령 파라미터(ASCII문자)에서 delay, 향 번호, 시간 읽어 오고  
// 향 분사 상태 플래그 'R' 세팅 후 실제 분사 루프(`DoScentBackLoop()`) 호출

- `RunDelayTime`: 향 발향 전 대기 시간 (ms 단위?)
    
- `RunScentNo`: 발향할 향 번호 (예: 1, 2, 3 ...)
    
- `RunPieriod`: 발향 지속 기간 (초 단위)
    

함수는 각 데이터를 아스키 코드 문자에서 숫자로 바꾸기 위해서 `atol()`, `atoi()` 함수를 씁니다.

## Run Clear 명령 (세정 명령)

- 위와 비슷하지만 'C' 플래그 세팅 후 세정 루프(`DoClearBackLoop()`) 실행
    

## Run Stop 명령 (발향 중지)

- 'S' 플래그 세팅 후 정지 루프(`DoStopBackLoop()`) 실행
    

# 7. 명령 처리 후 PC에 상태/버전 응답 송신 예

- `RunSerialHostCommandGetStatus()`
    
    - 내부 상태 데이터(`RunScentStatus` 배열)를 패킷에 넣어 ASCII 형태로 전송
        
- `RunSerialHostCommandGetVersion()`
    
    - 소프트웨어 버전 문자열을 패킷에 넣어 전송
        

송신은 `SerialHostSendData()`가 RS485 트랜시버를 활성화 시킨 후, 버퍼에 있는 데이터를 한 바이트씩 전송합니다.

# 8. 체크섬 처리 (XOR 방식)

- 수신된 데이터에 XOR 연산을 해 두 바이트(상위 4비트, 하위 4비트)를 16진수 ASCII 문자로 변환하여 데이터 뒤에 붙입니다.
    
- PC와 MCU 모두 이 체크섬으로 데이터 무결성 검증
    

# 9. 인터럽트 루틴

- `ISR(USART0_RX_vect)` → 시리얼 데이터 수신 시 호출
    
    - UART 상태 체크 후 버퍼에 깨진 데이터 없이 잘 저장 및 처리 함수 호출
        
- `ISR(USART0_UDRE_vect)` → 송신 버퍼 비었을 때 호출
    
    - 남은 데이터 전송 처리, 모두 보내면 인터럽트 중지
        

# 10. 정리와 이해를 돕는 비유

|단계|목표/역할|비유|
|---|---|---|
|데이터 수신|시리얼로 바이트 모으기|우표 하나하나 받는 우체국 직원|
|패킷 구분 및 완성|STX부터 ETX까지 한 덩어리 데이터 완성|편지 봉투 열기 (완전한 편지인지 확인)|
|명령 확인 및 파싱|명령 구분, 파라미터 분리|편지 내용 읽고, 명령서 분류|
|명령 실행 함수 호출|분사, 세정, 정지 등 기능 실행|공장 작업 지시서에 따라 기계 작동 시작|
|상태나 버전 송신|명령 처리 후 결과 보고|작업 완료 보고서 작성 및 전달|
|통신 오류 체크섬|데이터가 깨졌는지 검사|전달받은 편지가 훼손됐는지 점검|
|인터럽트|수시로 들어오는 데이터 관리|작업장 내 직원이 편지를 시간마다 체크 및 처리|

필요한 경우,

- 각 명령 처리 루프 (`DoScentBackLoop()`, `DoClearBackLoop()`) 동작 방식,
    
- 정확한 파라미터 포맷(데이터 자리수, 의미),
    
- 송수신 패킷 예시(16진수 시퀀스),
    
- RS485 통신 하드웨어 제어,
    
- MCU 레지스터 설정 등도 쉽게 설명해 드릴 수 있습니다.
    


---


# 1. include 파일 설명





```cpp

#include          
// 다양한 크기의 정수타입 정의 (uint8_t, int16_t 등)

#include    
// 하드웨어 제어(PWM, UART 등) 구현용, Arduino/AVR용 내부 함수

#include "IODefine.h"
// 입출력 핀 정의 (예: 센서, 벨브 연결 핀번호)

#include "nfserial.h"
// 시리얼 통신(NFSerial 클래스) 정의 #include "IOControl.h"

#include "IOControl.h" 
// 입출력 제어 함수 (핀 ON/OFF 등) #include 

#include "HostRun.h"
// Host 명령상수, 데이터구조, 함수 원형(프로토타입) 정의 #include "ScentControl.h" 

#include "ScentControl.h" 
// 향/세정 등 디바이스 동작 함수 정의

```
- 표준 및 프로젝트용 헤더파일을 모두 포함
    
- 시리얼 통신, IO제어, 프로젝트별 커맨드와 동작 등 모든 기반 코드 포함
    

# 2. 외부 변수 및 주요 구조체 의미

```cpp

extern NFSerial SerialDebug;      // 디버그용 시리얼. 로그, 상태 찍을 때 사용

  

NFSerial SerialHost(&UBRR0H, &UBRR0L, &UCSR0A, &UCSR0B, &UCSR0C, &UDR0);

// 메인 통신(Host↔장치)용 시리얼 객체. (AVR UART0 레지스터 직접 지정)

  

extern int MyAddress;             // 이 장치의 주소 (2글자. 예: '01')

extern int RecvFlag;              // 데이터 수신 여부 플래그

extern char RunScentStatus[4];    // 장치 동작상태(4byte) - 향/세정 등 각 채널상태

```

# 3. 프로토콜 구조 & 명령 리스트

```cpp

HostCommandDataTag HostCommand[] =

{

    { 'C', 7 + 0,   0 },     // 'C': 통신확인(Cofirm) - 데이터 없음

    { 'R', 7 + 13,  13 },    // 'R': 향분사(Run Scent) - Delay, Number, Period 등 13byte

    { 'L', 7 + 13,  13 },    // 'L': 세정(Run Clear)

    { 'S', 7 + 13,  13 },    // 'S': 중지(Run Stop)

    { 'G', 7 + 0,   0 },     // 'G': 상태요청(Get Status)

    { 'V', 7 + 0,   0 },     // 'V': 버전요청(Get Version)

    { 'Z', 7 + 17,  17 },    // 'Z': 밸브 제로브로드캐스트

    { 'z', 7 + 9,    9 },    // 'z': 밸브 제로설정

    { 'v', 7 + 0,    0 },    // 'v': 밸브상태요청

};

```

- `Command`: 명령 한 글자
    
- `Length`: 전체 명령길이
    
- `DataLength`: 데이터부분 길이(파라미터 길이)
    

# 4. 상수 정의 (프로토콜 제어문자, 버퍼사이즈 등)

```cpp

#define SERIAL_HOST_COMMAND_POS    3   // 명령위치 (시작, 주소, 주소, [명령])

#define SERIAL_HOST_COMMAND_LEN    1

#define SERIAL_HOST_PARAM_POS      4   // 데이터 시작위치

  

#define SERIAL_STX   0x02   // 시작문자

#define SERIAL_SOH   0x40   // 전송시작(송신용)

#define SERIAL_ETX   0x03   // 종료문자

#define SERIAL_ACK   0x06   // 통신성공(Ack)

#define SERIAL_NACK  0x15   // 통신실패(Nak)

  

#define SERIAL_HOST_BUFF_SIZE 128    // 시리얼 버퍼크기 128바이트

  

#define SerialHostTimeOutDefault 400 // 타임아웃카운트(40*10ms=4초)

  

//--- 송수신용 버퍼와 상태변수들

unsigned char   SerialHostRxBuff[SERIAL_HOST_BUFF_SIZE];        // 수신 버퍼

unsigned char   SerialHostRxCommandBuff[SERIAL_HOST_BUFF_SIZE]; // 명령 수신 버퍼

unsigned char   SerialHostRxCommandParam[SERIAL_HOST_BUFF_SIZE];// 명령 데이터 파라미터

unsigned int    SerialHostRxCount = 0;                         // 수신된 바이트 수

unsigned int    SerialHostRxCommandBuffCount = 0;              // 명령버퍼 바이트수

unsigned int    SerialHostCommandIndex = 0;                    // 명령 인덱스(HostCommand 테이블 기준)

unsigned char   SerialHostCommandActiveFlag = 0;               // 명령처리유무

unsigned char   SerialHostTxBuff[SERIAL_HOST_BUFF_SIZE];        // 송신 버퍼

unsigned int    SerialHostTxCount = 0;                         // 송신한 바이트 수

unsigned char   Host_CommandFlag;                              // 명령플래그(1이면 명령 도착, 처리대기)

unsigned char   Host_SerialRunFlag;                            // 수신시작플래그(패킷 처음부터 수신 중)

  

int SerialHostTimeOutCount = SerialHostTimeOutDefault;         // 타임아웃 카운트

```

- **패킷별 송수신을 위한 버퍼, 길이 저장 변수들이 반복적으로 쓰임**
    
- 각종 플래그 변수(패킷완결, 시작 등)로 명확한 흐름 컨트롤
    

# 5. 주요 함수 해설 (하나씩!)

## 5.1. 시간, 타임아웃 함수

cpp

`void SerialHostTimeOutCheck() // 주기적으로 실행(50ms마다) {     if (SerialHostTimeOutCount)        SerialHostTimeOutCount--; // 타임아웃카운트 감소 (0되면 타임아웃) } char IsSerialHostTimeOut() {     if (SerialHostTimeOutCount == 0)        return 1;    else        return 0; }`

- **명령 실행 중 일정 시간(4초 정도)이 지난 후 응답 없으면 실패판정**
    
- 상태 관리, 안전장치 역할
    

## 5.2. 패킷수신, 명령파싱(SerialHostEvent, RunSerialHostDataCheck)

cpp

`// 시리얼 수신 인터럽트/루프마다 실행: 들어온 바이트 읽고 패킷모으기 void SerialHostEvent(void) {     int C;    while (SerialHost.available())    {        C = SerialHost.read();    // 1바이트 읽기        if (C == -1)            return;        if (C == SERIAL_STX)     // 시작문자 오면        {            memset(SerialHostRxBuff, 0, sizeof(SerialHostRxBuff));            SerialHostRxCount = 0;            Host_SerialRunFlag = 1;        }        if (Host_SerialRunFlag)        {            SerialHostRxBuff[SerialHostRxCount++] = C; // 버퍼에 저장            if (C == SERIAL_ETX)        // 끝문자 오면 패킷 완성                Host_CommandFlag = 1;            if (SerialHostRxCount >= sizeof(SerialHostRxBuff))            {   // 버퍼 오버플로우 - 초기화                memset(SerialHostRxBuff, 0, sizeof(SerialHostRxBuff));                SerialHostRxCount = 0;                Host_SerialRunFlag = 0;                Host_CommandFlag = 0;            }        }    } }`

cpp

`// 패킷 완성시 명령 구분 및 데이터 추출 void RunSerialHostDataCheck(void) {     int n;    if (Host_CommandFlag == 0)        return;    memcpy(SerialHostRxCommandBuff, SerialHostRxBuff, SerialHostRxCount);    SerialHostRxCommandBuffCount = SerialHostRxCount;    memset(SerialHostRxBuff, 0, sizeof(SerialHostRxBuff));    SerialHostRxCount = 0;     // 명령 문자(Command)와 HostCommand테이블의 n번째 명령 매칭    for (n = 0; n < (sizeof(HostCommand) / sizeof(HostCommandDataTag)); n++)    {        if ((SerialHostRxCommandBuff[SERIAL_HOST_COMMAND_POS] == HostCommand[n].Command))            break;    }     if (n < (sizeof(HostCommand) / sizeof(HostCommandDataTag)))    {   // 유효 명령이면 파라미터부분 추출        memset(SerialHostRxCommandParam, 0, sizeof(SerialHostRxCommandParam));        memcpy(SerialHostRxCommandParam, &SerialHostRxCommandBuff[SERIAL_HOST_PARAM_POS], HostCommand[n].DataLength);        SerialHostCommandIndex = n;        SerialHostCommandActiveFlag = 1;    }    else    // 알 수 없는 명령!    {        SerialHostCommandIndex = -1;        SerialHostCommandActiveFlag = 0;    }    RunSerialHostCommand();      // 실제 명령 수행    Host_CommandFlag = 0;        // 처리완료 }`

- **수신→임시버퍼 저장→명령 구분→파라미터 분리→명령 실행 흐름**
    
- 모든 명령패킷 공통 처리법
    

## 5.3. 시리얼 인터럽트 핸들러

cpp

`ISR(USART0_RX_vect) // 시리얼 수신완료 인터럽트 {     if (bit_is_clear(UCSR0A, UPE0))    {        uint8_t c = UDR0;        SerialHost._rx_complete_irq(c);        SerialHostEvent();    }    else    {        uint8_t c = UDR0;    } } ISR(USART0_UDRE_vect) // 송신완료 인터럽트 {     if (SerialHost.Txavailable())    {        SerialHost._tx_udr_empty_irq();        SerialHost._TxEndFlag = 0;    }    else    {        if (SerialHost._TxEndFlag == 0)        {            cbi(UCSR0B, UDRIE0);            SerialHost._TxEndFlag = 1;        }    } }`

- **수신/송신 인터럽트마다 시리얼 데이터 입출력 동작**
    
- 하드웨어 레벨, 프레임 하나하나 안정적으로 처리
    

## 5.4. 데이터 송신

cpp

`void SerialHostSendData(unsigned char* p, int Count) {     int i;    RS485_Tx_Enable();    delay(4);    for (i = 0; i < Count; )    {        if (SerialHost.availableForWrite())        {            SerialHost.print((char)*p++);            i++;        }    } }`

- RS485 송신모드 전환→데이터 한 글자씩 송신
    
- availableForWrite로 버퍼초과 방지
    

## 5.5. 체크섬 처리

cpp

`int SerialCheckCheckSum(unsigned char* p, int Count) {     // Count 바이트까지 XOR로 CHK값 만든 뒤 (16진수 → 문자)    // 뒤 2글자는 이 값이 맞는지 확인 } int MakeSerialCheckSum(unsigned char* p, int Count) {     // CHK 계산 후 p[Count]=CheckH, p[Count+1]=CheckL, 종료문자까지 추가, 전체길이 반환 }`

- **데이터 무결성 검사용 XOR 체크섬**
    
- HEX 값 2글자 문자로 붙임
    

## 5.6. 각 명령별 실행함수

(아래는 일부만 예시로, 나머지도 매우 유사 구조)

cpp

`int RunSerialHostCommandConfirm() {     // Reply 패킷 생성 ("Confirm OK")    // 체크섬 붙이고 송신    // 코드 흐름 동일 } int RunSerialHostCommandRunScent() {     // 명령 데이터에서 Delay, ScentNo, Period 읽어서 전역 변수 세팅    // 향 분사 함수(DoScentBackLoop) 실행    // 결과 요청 } int RunSerialHostCommandRunClear() {     // 명령 데이터에서 Delay, ScentNo, Period 읽어서 세정 명령 수행    // 세정 함수, 상태 송신 등 } int RunSerialHostCommandGetStatus() {     // 현재 상태를 읽어서 패킷 포맷으로 송신 } int RunSerialHostCommandGetVersion() {     // 버전정보 문자열 생성, 패킷 송신 }`

- **모든 명령은 패킷 처리 → 해당 동작 함수 호출 → 결과 송신**의 일관된 구조
    

# 6. 명령 패킷의 전체 수행 흐름

1. 시리얼 데이터 수신(인터럽트, Event())
    
2. 패킷 완성(STX~ETX 수집), 명령 파싱(RunSerialHostDataCheck)
    
3. 올바른 명령이면 데이터 분리, 지정 함수(RunSerialHostCommand) 실행
    
4. 연산(DoScentBackLoop 등) & 결과 송신 패킷 전송
    
5. 진행상황/에러/버전정보 등 응답
    

# 7. HostRun.h 헤더파일 내용 설명[1](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/02797fa0-df22-47ce-9161-450fcf41b5b0/HostRun.h)

- **NFSerial SerialHost**: Host(PC 등)와 통신하는 메인 시리얼 객체(여기서 전역선언)
    
- **RunDelayTime, RunPieriod, RunScentNo, RunScentClearFlag**: 분사 대기시간, 향번호, 주기, 동작종류('R'향/'C'세정등) 등 제어용 변수
    
- **HostCommandDataTag 구조체**: 패킷 명령에 대한 정보 저장  
    (명령문자, 전체패킷 길이, 데이터부분 길이)
    
- **enum 명령리스트**: 코드에서 명령번호 사용 시 매칭용
    
- **모든 함수 원형 선언**: .c 내부 함수들 미리 나열(컴파일러용)
    
- **타임아웃 함수, 각 명령별 실행함수(Confirm, RunScent 등) 원형**
    

# 정리

- **시리얼로 들어온 모든 명령을 STX~ETX까지 패킷단위로 처리**
    
- **각 명령에 따라 데이터를 분리하고, 동작 분기**
    
- **동작 시 Delay, Number, Period 등 항상 파싱/적용**
    
- **안전하게 버퍼 로직, 플래그, 타임아웃 등 견고하게 짜여 있음**
    
- **직접 하드웨어/펌웨어와 연동하여 임베디드에서 그대로 동작하는 구조**
    

궁금한 각 명령/상세 변수 해설이나, 실제 데이터 송수신 예시가 더 필요하면 언제든 요청하세요!

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/02797fa0-df22-47ce-9161-450fcf41b5b0/HostRun.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/02797fa0-df22-47ce-9161-450fcf41b5b0/HostRun.h)