

# 전반적 개요

- **구성 모듈 및 라이브러리**
    
    - `<rs232.h>`, `<utility.h>`, `<formatio.h>` 같은 LabWindows/CVI 라이브러리로 시리얼 통신, 문자열 입출력 등을 처리
        
    - `ControlPumpNF.h`, `ConfigCOM.h`, `FunctionSerial.h`는 프로젝트별 헤더로, 시리얼 구조체 등 정의 포함 예상
        
- **핵심 구조**
    
    - `struct s_SERIAL` 타입 변수(`scom`)에 시리얼 포트 설정, 송수신 데이터 등이 포함
        
    - 설정파일(`ConfigCom.cfg`)을 읽어 시리얼 통신 초기 세팅값을 로드
        
    - 포트 열고, 통신 시간 설정, 콜백 등록(쓰기 및 읽기 이벤트 감지) 수행
        
    - 송신 시 CRC를 붙이거나 프로토콜(SLIP, MODBUS 등) 인코딩 처리 후 전송
        
    - 수신 시 처리함수 `ReadFromCom`가 호출되어 데이터 읽고 후처리
        

# 주요 함수 및 역할

## 1. `ComCallback(int port,int eventMask,void *dataq)`

- 시리얼 이벤트가 발생할 때 호출되는 콜백 함수
    
- 정상 수신 등 이벤트 시 `ReadFromCom(&s_serial)` 호출하여 수신 처리 시작
    

## 2. `ReadConfigCom(struct s_SERIAL *scom)`

- **설명**: `ConfigCom.cfg` 파일을 열어 시리얼 포트 설정 읽어 에 구조체(`scom`)에 저장
    
- 줄 단위로 읽어 필요한 설정값들(포트 번호, baudrate, parity, databits, stopbits, timeout, protocol, terminator, CRC, mode 등)을 `Scan()`으로 파싱
    
- timeout 단위는 밀리초 → 초 단위로 변환 저장
    
- 설정파일 이 없거나 열지 못할 경우엔 함수 종료
    

## 3. `ActivateCom(struct s_SERIAL *scom)`

- 시리얼 포트 오픈 및 초기화 실시
    
- `OpenComConfig()` 함수에 시리얼 설정값('포트번호', '속도', '패리티' 등) 전달해 실제 포트 오픈
    
- 성공시 송수신 큐 초기화(`FlushInQ`, `FlushOutQ`)
    
- 타임아웃 설정 및 이벤트 콜백 등록(`InstallComCallback`)
    
- 실패 시 `ErrorCom()` 호출해 에러 처리
    

## 4. `SendToCom(struct s_SERIAL *scom)`

- 송신 데이터가 담긴 `scom->send` 버퍼에 CRC 등을 첨부하고 프로토콜 인코딩 적용
    
- CRC 종류에 따라 `AttachCRC16`, `AttachModbusCRC` 호출하여 CRC 계산 후 추가
    
- 프로토콜에 맞게 종료문자 첨가 또는 SLIP 인코딩 수행
    
- 디버그 레벨에 따라 전송 데이터를 16진수 혹은 문자열 형태로 출력
    
- 실제로는 `ComWrt()`로 시리얼 포트로 데이터 전송
    
- 오류 발생 시 -1 리턴
    

## 5. `ReadFromCom(struct s_SERIAL *scom)`

- 수신 데이터를 읽는 함수 (부분 코드만 제공됨)
    
- 통신 종료문자(terminator)가 설정되면, 그 문자까지 데이터 수신을 시도하고 처리
    

# 데이터 흐름 정리

text

`시작 → ReadConfigCom()로 설정 로딩 → ActivateCom()으로 포트 오픈 및 콜백 등록       
↓  
scom->send에 데이터 세팅 → SendToCom() 호출하면 CRC첨부, 프로토콜 인코딩 → 실제 COM포트로 전송      
↓  
시리얼 포트에 수신 데이터 들어오면 콜백 함수 ComCallback() 자동 호출      
↓  
ComCallback()에서 ReadFromCom() 호출 → 수신 데이터 읽고 후처리`

# 동작 원리 및 참고사항

- **콜백 기반 통신**:  
    이벤트 발생 기반 비동기 입출력 처리 → CPU 계속 대기하지 않고도 효율 통신 가능
    
- **프로토콜 및 CRC 처리**:  
    기본 RS232는 오류검출 없으므로 CRC 붙이거나 Modbus, SLIP 프로토콜 인코딩을 통해 안정성 확보 가능
    
- **설정파일 관리**:  
    포트 설정과 운영 방식을 CFG파일로 외부 관리해 동적 설정 편리
    

# 추가 도움

- `struct s_SERIAL` 멤버 구성 상세
    
- `AttachCRC16`, `AttachModbusCRC`, `EncodeSLIP` 함수 내부 원리
    
- `OpenComConfig()`, `InstallComCallback()`, `ComWrt()` 등 라이브러리 함수 상세 활용법
    
- 프로토콜별 송수신 패킷 구조 해석
    
- 에러 처리 및 디버깅 팁
    

필요하시면 위 내용 중 특정 항목 또는 소스코드에 대한 더 구체적 설명 부탁해 주세요!