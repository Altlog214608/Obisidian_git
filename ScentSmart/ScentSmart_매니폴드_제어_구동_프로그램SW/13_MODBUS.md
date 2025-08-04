주신 MODBUS 관련 파일은 **Modbus 프로토콜** 명령을 PC 소프트웨어에서 만들어 하드웨어(펌웨어) 장치에 보내는 함수들과 관련 정의를 포함하고 있습니다.  
이 내용을 초심자도 이해할 수 있도록 쉽고 자세하게 정리해 드리겠습니다.

# MODBUS 파일 개요

- **목적** :  
    PC 애플리케이션에서 사용하는 Modbus 통신 명령 생성 함수들이며,  
    보통 RS-232, RS-485 같은 시리얼 통신으로 펌웨어 장치와 데이터 주고 받는 데 쓰입니다.
    
- **Modbus 프로토콜 배경**:
    
    - 산업용 표준 통신 프로토콜 중 하나로,
        
    - 장치의 디지털/아날로그 상태 레지스터를 읽고 쓰는데 쓰임
        
    - 명령과 데이터가 정해진 규격에 따라 바이트 배열로 전송됨
        

# 1. 매크로 정의



```c
#define MBC_RCL 0x01 
// Read Coils (비트 읽기) 
#define MBC_RDI 0x02 // Read Discrete Inputs (비트 읽기) 
#define MBC_RHR 0x03 // Read Holding Register (16비트 읽기/쓰기 가능) 
#define MBC_RIP 0x04 // Read Input Register (16비트 읽기 전용) 
#define MBC_WSC 0x05 // Write Single Coil (비트 쓰기) 
#define MBC_WSR 0x06 // Write Single Register (16비트 쓰기) 
#define MBC_WMC 0x0F // Write Multiple Coils (비트 여러개 쓰기)
#define MBC_WMR 0x10 // Write Multiple Registers (16비트 여러개 쓰기)
```

- Modbus 통신 시에 **명령 코드(Command Code, Function Code)** 로 사용됩니다.
    
- 예컨대,
    
    - `0x03`은 16비트 홀딩 레지스터 읽기용 명령,
        
    - `0x06`은 16비트 홀딩 레지스터 쓰기용 명령 코드입니다.
        

# 2. 주요 함수

## 2-1. `ReadRegisters_MODBUS`



```c
void ReadRegisters_MODBUS(unsigned char readcmd, unsigned short add, unsigned short qtres)
```

- **역할** :  
    Modbus의 “레지스터 읽기” 명령을 만들어 `s_serial.send` 버퍼에 넣고,  
    `SendToCom()` 함수를 호출해 장치에 전송합니다.
    
- **입력 파라미터**:
    
    - `readcmd`: 읽을 레지스터 종류 명령(예: `MBC_RHR(0x03)`, `MBC_RIP(0x04)`)
        
    - `add`: 시작 레지스터 주소 (예: 0x0FA0 등)
        
    - `qtres`: 읽을 레지스터 개수 (quantity of registers)
        
- **내부 동작** :  
    `s_serial.send[]` 배열에 통신 패킷 데이터 바이트 순서대로 저장
    
    text
    
    `send[0] = 명령코드 send[1] = 주소 high byte (예: 상위 8비트) send[2] = 주소 low byte send[3] = 읽을 레지스터 수 high byte send[4] = 읽을 레지스터 수 low byte`
    
- `s_serial.sendbyte = 5`로 총 바이트 수 지정 후 전송
    
- **즉,**  
    Modbus 명령 프레임에서 다음과 같은 구조를 가진 부분을 보냅니다
    
    |바이트 위치|내용|
    |---|---|
    |0|Function Code (Read command)|
    |1-2|Register Start Address (2 bytes)|
    |3-4|Number of Registers to Read (2 bytes)|
    

## 2-2. `WriteSingeRes_MODBUS`



```c
void WriteSingeRes_MODBUS(unsigned short add, unsigned short value)
```

- **역할**:  
    Modbus 단일 레지스터 쓰기(Write Single Register, command 0x06) 명령을 작성해 보내는 함수
    
- **입력 파라미터** :
    
    - `add`: 쓰려는 레지스터 주소 (16비트)
        
    - `value`: 쓸 데이터 값(16비트)
        
- **구성**:
    
    text
    
    `send[0] = MBC_WSR (0x06) send[1] = 주소 상위 바이트 send[2] = 주소 하위 바이트 send[3] = 값 상위 바이트 send[4] = 값 하위 바이트`
    
- 총 5바이트 전송 설정
    

# 3. 사용 예와 이해

- 만약, 16비트 홀딩 레지스터 주소 0x1068에 값 0x0002 쓰고 싶으면,
    



```c
WriteSingeRes_MODBUS(0x1068, 2);
```

- Modbus 명령 프레임 일부가 아래와 같이 `s_serial.send` 배열에 채워지고,
    
- `SendToCom`으로 실제 RS232/RS485 포트 전송
    

# 4. 상위 프로토콜과 통신 방식

- 이 함수들은 **Modbus 명령 프레임의 일부만을 생성**
    
- 실제 통신 데이터는 `SendToCom()` 등에서 CRC, 주소 ID, 시작/종료 등 패킷 전체 구조를 더해 완전한 형태로 전송됨
    
- 복수 레지스터 쓰기, 읽기 등은 별도 함수(`WriteMultiRes_NF` 등)에서 처리
    

# 5. 정리

|항목|설명|
|---|---|
|`MBC_RHR=0x03`|Modbus 읽기(홀딩 레지스터) 명령 코드|
|`ReadRegisters_MODBUS`|Modbus 읽기 명령 생성 (시작 주소, 개수 포함)|
|`WriteSingeRes_MODBUS`|Modbus 단일 레지스터 쓰기 명령 생성|
|`s_serial.send`|시리얼 송신용 데이터 버퍼 (명령 바이트 배열)|
|`sendbyte`|보낼 바이트 총 수 (버퍼 길이)|
|`SendToCom()`|실제 시리얼로 명령 전송 함수|

이해하신 기존 통신 구조(`ScentSmart.py` → `HostRun.cpp` 등)와 연결하면,

- PC에서 Modbus 명령을 이런 식으로 패킷 빌드 → RS485 시리얼 전송 → MCU에서 명령 해석 → 장비 제어 되는 구조입니다.
    

필요하면, 관련 연동 예시나 송수신 패킷 구조를 실제 16진수 시퀀스로 보여드릴 수도 있습니다.  
궁금하시면 말씀해 주세요!