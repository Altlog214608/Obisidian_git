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
