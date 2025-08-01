아래는 주신 LabWindows/CVI 기반 C 소스코드와 관련 헤더를 쉽고 자세하게 풀어서 설명한 내용입니다.  
주로 RS-232/시리얼 통신, Modbus 프로토콜 패킷 생성과 처리, UI 이벤트 처리와 연동 등을 다룹니다.

# 1. 프로그램 개요

- LabWindows/CVI 환경에서 RS-232 시리얼 통신을 통해 하드웨어(펌웨어 장치)와 데이터 송수신하는 프로그램 소스입니다.
    
- 사용자 인터페이스(UI)를 띄워 장비 제어 및 상태 모니터링 수행.
    
- 설정파일에서 COM포트 등 시리얼 설정을 읽고 초기화하며, 통신 프로토콜(특히 Modbus) 기반 명령 및 데이터 송수신 구현.
    

# 2. 주요 파일 및 역할

|파일(헤더)|역할 설명|
|---|---|
|ConfigCom.h|시리얼 포트 설정, 구조체 및 관련 함수 선언|
|ControlPumpNF.h|하드웨어 펌프 관련 명령 및 데이터 주소 정의|
|MODBUS.h|(예상) Modbus 관련 상수 및 프로토콜 정의|
|FunctionSerial.h|시리얼 송수신 관련 함수 구현|
|uc_control_level_32.h|UI 패널 빌드 함수 선언 (CVI UI 관련)|

# 3. `main` 함수 및 UI 초기화 흐름

c

`if (InitCVIRTE(0, argv, 0) == 0) return -1;  GetProjectDir(proj_dir);`

- CVI 런타임 환경 초기화
    
- 현재 프로젝트 폴더 경로를 얻음
    

c

`control_handle = BuildCPL(0); acq_handle = BuildDPNL(0);`

- UI 패널 빌드 함수 호출 (CVI UI 런타임용 생성)
    
- 예전 코드에서 `LoadPanel`로 .uir 파일을 직접 로드하던 부분은 주석 처리됨
    

c

`DisplayPanel(control_handle); DisplayPanel(acq_handle); SetCtrlAttribute(control_handle, CPL_QUIT, ATTR_VISIBLE, 0);`

- 각 패널 창을 화면에 띄우고, 종료 버튼 숨김 처리
    

c

`InitProcess(); RunUserInterface();`

- 통신 및 기타 초기화 수행 후 UI 루프 실행
    

# 4. 통신 초기화 및 설정 (`InitProcess`)

c

`ReadConfigCom(&s_serial); ActivateCom(&s_serial); InitProcess_NF();`

- `s_serial` 구조체에 시리얼 설정파일(`ConfigCom.cfg`)을 읽어 설정값 로딩
    
- 시리얼 포트를 열고 활성화
    
- 하드웨어 관련 초기 읽기/쓰기 명령 (펌웨어 버전 조회, 펌프 컨트롤 선언 등) 실행
    
- 주기적 읽기용 타이머 활성화
    

# 5. 시리얼 패킷 송수신 함수 개요

## `WriteSingleRes_NF`

- 하나의 레지스터에 단일 데이터 기록하는 Modbus 명령 작성
    

c

`s_serial.send[index++] = NF_MID;  // ID s_serial.send[index++] = MBC_WSR; // Write Single Register 커맨드 s_serial.send[index++] = (add>>8) & 0xFF;                // 레지스터 주소 상위 1바이트 s_serial.send[index++] = add & 0xFF;                      // 하위 1바이트 s_serial.send[index++] = (value>>8) & 0xFF;              // 값 상위 1바이트 s_serial.send[index++] = value & 0xFF;                    // 값 하위 1바이트`

- `sendbyte`에 총 길이 설정 후
    
- `SendToCom()`으로 전송, `ReadFromCom()`으로 응답을 즉시 읽음
    

## `ReadRegisters_NF`

- 여러 레지스터를 읽는 Modbus 읽기 명령 작성
    

c

`s_serial.send[index++] = NF_MID;  // ID s_serial.send[index++] = readcmd;  // 명령 (e.g. Read Input Register) s_serial.send[index++] = (add>>8) & 0xFF;  // 읽을 레지스터 주소 상위 s_serial.send[index++] = add & 0xFF;        // 하위 s_serial.send[index++] = (qtres>>8) & 0xFF;  // 읽을 레지스터 개수 상위 s_serial.send[index++] = qtres & 0xFF;        // 하위`

- 이후 `sendbyte` 길이 설정 → 송신, 수신 처리
    

## `WriteMultiRes_NF`

- 여러 레지스터에 여러 값(복합 명령 데이터) 쓰기 명령 작성
    

주요 특징:

- 명령 종류, 향 번호, 펌프 파워(Linear 변환 적용), 런타임, 딜레이 등 여러 데이터를 한 번에 전송함
    
- 파워 레벨은 아래 변환식으로 값을 조절하여 펌프에 맞춤
    

c

`gain = (double)(80-30)/(31-1); offset = (double)(30 - gain); cpower = (int)(gain * s_scent.level + offset + 0.5);`

- 이 `cpower` 값은 고수준 UI에서 설정한 값(레벨)에 대응하며, 펌프 출력에 맞는 %값으로 변환
    
- `send` 배열에 차례대로 바이트를 넣고 총 길이 설정 후 송신과 수신 처리
    

# 6. 오류 처리 및 디코딩 (`Error_NF`, `DecodePacket`)

## `Error_NF(int errindex)`

- 시리얼 패킷 해석 중 발생한 에러코드에 대응하는 오류메시지를 보여줌
    
- 예: COMMAND_ERROR, LENGTH_ERROR, CHECKSUM_ERROR, SCENT_COMMAND_RANGE_OVER 등
    
- UI 내 에러 표시 컨트롤에 메시지 띄우고 LED 상태 등 설정
    

## `DecodePacket()`

- 수신한 패킷을 해석해,
    
- 에러 플래그 검사 후 에러 출력 또는 정상 상태 표시
    
- 특정 읽기 명령(MBC_RIP, MBC_RHR)일 때, 읽은 레지스터값을 숫자로 변환해 문자열 빌드 및 UI(레이블, 그래프) 출력
    

# 7. UI 콜백 함수

- UI 버튼이나 타이머 이벤트 발생 시 동작 함수
    

## `C_RunProcess`

- 커맨드, 향 번호, 세기, 시간 등을 UI 컨트롤에서 읽어서
    
- cmdid에 대응하는 함수 호출 (예: run, clean, stop 명령 전송)
    

## `C_StopProcess`

- 정지 명령 발송(WriteMultiRes_NF with cmdid=3)
    

## `C_QuitProcess`

- 프로그램 종료 처리
    

## `C_Readdata`

- 타이머 이벤트 주기로 센서 데이터 읽기 명령 실행
    

# 8. 핵심 데이터 정의(MODBUS 주소)

c

`#define NF_MID 0x01   // 장치 ID // 읽기 레지스터 주소 예시 #define NF_FVM 0x0FA0  // 펌웨어 버전 #define NF_CGP 0x0FCD  // 캐리어 가스 압력 센서 // 쓰기 레지스터 주소 #define NF_CMD 0x1068  // 명령 (1:run, 2:clean, 3:stop) #define NF_BNO 0x1069  // 병(향) 번호 #define NF_RPW 0x106A  // 런 펌프 파워 % #define NF_CPW 0x106B  // 세정 펌프 파워 % #define NF_POT 0x106C  // 펌프 웨이트(초) #define NF_COT 0x106D  // 세정 타임 (msec)`

- UI → MCU 명령 및 센서 데이터 모두 이 주소들을 기준으로 주고받음
    

# 9. 통신 흐름 간단 요약

text

`1) UI에서 cmdid, 향 번호, 런타임 등 설정 → WriteMultiRes_NF 호출 2) send 버퍼에 Modbus 패킷 데이터 바이트 단위 작성 3) SendToCom() 호출하여 CRC 등 프로토콜 추가 후 RS232 전송 4) ReadFromCom() 호출하여 MCU 응답 수신 및 UI 업데이트 5) 주기적 센서 데이터 읽기(ReadRegisters_NF)와 처리 6) 에러 발생 시 Error_NF() 호출해 UI 표시`

# 요약 및 차이점

|이번 소스 (ControlPumpNF 등)|이전 ScentControl / HostRun MCU 코드|
|---|---|
|PC 측 LabWindows/CVI C 코드 구현|MCU 내 C/C++ 펌웨어 하드웨어 제어 구현|
|RS-232 COM 포트 초기화, 이벤트 등록 등|인터럽트 기반 UART 수신 송신 처리|
|Modbus 프로토콜 기반 송수신 명령 조립 및 해석|펌웨어 내 명령 해석, 솔레노이드 밸브 제어 루프 및 타이머 관리|
|UI 패널 제어 및 이벤트 처리|MCU 제어 로직, GPIO, PWM 제어 등 펌웨어 하드웨어 처리 코드|

필요하시면

- 각 함수 내부 상세 코드 흐름
    
- 송·수신 데이터 실제 예시(바이트값)
    
- CRC, SLIP, MODBUS 프로토콜 처리과정
    
- IVI UI 변수와 컨트롤간 데이터 매핑
    
- 디버깅 팁 및 에러 처리 심화 내용  
    등도 자세히 알려 드릴 수 있습니다.  
    필요한 부분 말씀해 주세요!