MODBUS 프로토콜은 산업 자동화 분야에서 널리 사용되는 통신 규격으로, 장치 간의 마스터-슬레이브(master-slave) 방식의 통신 구조를 가집니다. 하드웨어(시리얼 라인, TCP/IP 등)와 무관하게, **데이터의 포맷, 명령 구조, 체크섬(CRC/LRC) 등 표준 구조**가 정의되어 있습니다.

## 1. 기본 개념

- **마스터(주장치)**가 명령(Request)을 보내고, **슬레이브(종속장치)**가 응답(Response)을 돌려줍니다.
    
- 대표적인 전송 방식: **RTU(바이너리, 16진수 프레임)**, **ASCII(ASCII 코드 프레임)**, **TCP/IP(이더넷용)** 등이 있습니다[1](https://www.modbustools.com/modbus.html)[2](https://www.waveshare.com/wiki/Modbus_Protocol_Specification)[3](https://unserver.xyz/modbus-guide/).
    

## 2. MODBUS RTU Frame(프레임) 구조

가장 많이 사용되는 **RTU(Serial, 바이너리 방식)**의 표준 프레임 구조는 다음과 같습니다:

|필드명|크기|설명|
|---|---|---|
|Slave Address|1 Byte|대상(슬레이브) 장치 주소(0~247)|
|Function Code|1 Byte|동작 명령 코드 (예: 0x03: 읽기)|
|Data|가변|옵션/파라미터 등 데이터(길이 가변)|
|CRC|2 Bytes|메시지 무결성 확인(16비트 CRC)|

예시(7바이트짜리 요청 프레임):

text

`[주소][함수][데이터][CRC Low][CRC High]  01    03    02 00 01  25     CA`

- **01**: 슬레이브 주소
    
- **03**: 함수코드(Read Holding Register)
    
- **02 00 01**: 데이터 및 파라미터(레지스터, 길이 등)
    
- **25 CA**: CRC16 체크섬[4](http://c5iot.com/protocols_rand_9527/EM735.pdf)[3](https://unserver.xyz/modbus-guide/)
    

## 3. 주요 필드 역할

- **Slave Address**: 명령 대상 장치(슬레이브) 지정
    
- **Function Code**: 수행할 동작 종류(읽기, 쓰기 등) 지정
    
- **Data**: 주소, 값, 데이터 길이 등 명령 세부 정보
    
- **CRC**: 프레임 내 모든 바이트(마지막 2바이트 CRC 제외)에 대해 16비트 CRC 계산 결과(Lo High 순)
    

## 4. CRC16 계산 방식

- CRC는 0xFFFF로 초기화한 16비트의 레지스터에 프레임 데이터를 1바이트씩 XOR, 비트 이동, 조건별 XOR(0xA001) 연산을 적용하며 마지막에 high/low 바이트 순서로 결과를 붙입니다[5](https://www.iotrouter.com/es/introduction-to-modbus-protocol-and-calculation-method-of-modbus-rtu-crc-check-code/)[6](https://ctlsys.com/support/how_to_compute_the_modbus_rtu_message_crc/)[7](https://github.com/LacobusVentura/MODBUS-CRC16).
    
- 데이터 무결성 확인용. 송신측/수신측 모두 같은 방식으로 계산하여 일치하지 않으면 에러로 간주합니다.
    

## 5. ASCII/TCP와 차이

- **ASCII 모드**: 각 바이트를 2자리 16진수 ASCII 코드로 인코딩, LRC(Longitudinal Redundancy Check) 사용
    
- **TCP/IP 모드**: 7바이트 MBAP 헤더(고유 식별자+길이 등)가 붙고, CRC 체크섬 대신 TCP/IP의 오류 감지를 이용함[2](https://www.waveshare.com/wiki/Modbus_Protocol_Specification)[8](https://ozeki.hu/p_5843-modbus-frame-format-types.html)[9](https://www.emqx.com/en/blog/modbus-protocol-the-grandfather-of-iot-communication).
    

## 6. 추가 설명

- **긱별 명령(함수 코드)**: 예) 0x03(Read Holding Registers), 0x06(Write Single Register), 0x10(Write Multiple Registers) 등등.
    
- **최대 프레임 길이**: RTU 기준 최대 256바이트(1+1+252+2)
    
- **동작 흐름**: 마스터 → 슬레이브로 Request 전송, 슬레이브에서 Response 회신
    

## 요약 예시 (RTU)

1. **마스터**가 [주소][함수][데이터][CRC] 구조 프레임 송신
    
2. **슬레이브**가 파싱/처리 후 응답 프레임 송신(동일 구조)
    
3. 에러 시 Exception Response 프레임 반환
    

**MODBUS 프로토콜**은 이처럼 간단·정형화된 프레임 구조로 데이터 및 명령 통신을 수행하며,  
CRC를 통해 데이터 무결성을 강하게 보장합니다.



## 1. 마스터-슬레이브 구조 명칭의 이유

- **마스터(Master)**:  
    항상 “주도적으로 명령(Request)”을 보내는 주체입니다.  
    어떤 데이터가 필요하거나 장치 제어(읽기/쓰기 등)를 요청할 때 먼저 명령을 내립니다.
    
- **슬레이브(Slave)**:  
    “명령에 응답(Response)”하는 역할로,  
    마스터가 보내는 요청에 대해서만 데이터를 전송하거나 요청받은 동작을 수행합니다.  
    슬레이브가 임의로 먼저 데이터를 보내는 방식은 아닙니다.
    
- **즉, 마스터가 질문/명령을 주도하고, 슬레이브는 그에 대한 답변/실행만 하는 구조**이기 때문에 이런 명칭이 붙었습니다.
    

## 2. 대표적인 MODBUS 전송 방식 3가지 비교

|전송 방식|특징/주요 용도|데이터 포맷|체크섬|네트워크/회선|
|---|---|---|---|---|
|**RTU**|가장 널리 쓰임. 이진(Binary) 프레임, 속도/효율 최상|Hex(바이너리)|CRC16|RS-232, RS-485 등|
|**ASCII**|사람이 읽기 쉬움, 디버깅 편리|7비트 ASCII(문자)|LRC(Longitudinal)|RS-232, 485|
|**TCP/IP**|이더넷 기반, 산업 자동화 네트워크에서 사용|TCP/IP 패킷(바이너리)|없음(TCP 체크)|Ethernet(Internet/LAN)|

## 각 방식 상세 설명

- **RTU (Remote Terminal Unit)**
    
    - 8비트 바이너리 데이터
        
    - 효율이 높고, 프레임이 짧으며, 산업 현장에 가장 많이 사용됨
        
    - CRC16 체크섬으로 오류 검출
        
- **ASCII**
    
    - 각 바이트를 두 자리 16진수 ASCII 문자로 변환하여 전송
        
    - 사람이 직접 읽고 디버깅하기 쉽지만, 데이터 효율(속도)은 떨어짐
        
    - LRC(Longitudinal Redundancy Check) 방식 오류 검출 사용
        
- **TCP/IP**
    
    - MODBUS 프로토콜을 이더넷 패킷(TCP) 상에서 동작하도록 만든 것
        
    - MBAP(Modbus Application Protocol) Header가 붙음, 주소 개념 다름
        
    - 라인 노이즈 없이, 대규모 네트워크에 적합
        

**정리:**

- 마스터/슬레이브 구조는 “명령자-응답자” 역할이 명확해서 이런 이름이 붙었고,
    
- MODBUS는 RTU(이진), ASCII(문자), TCP/IP(네트워크) 3가지 방식이 널리 쓰이며,  
    각각의 데이터 포맷·용도·체크섬 방식에 차이가 있습니다.
    

궁금한 점 추가로 말씀해주시면 더 구체적으로 안내해 드릴 수 있습니다!