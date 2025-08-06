좋아! 너가 이 시스템을 잘 이해하고 유지보수하거나 개선하려면 지금 언급한 기술들을 확실히 알아야 해. 아래는 **RS232, MODBUS RTU, CVI, CRC, SLIP 인코딩**에 대한 설명이야. **현 시스템 코드와 연관지어서** 최대한 쉽게 설명할게.

---

## ✅ 1. RS232란?

> **PC ↔ 하드웨어(펌프 등)** 간 **직렬 통신**(Serial Communication) 규격

### 🔧 특징

|항목|설명|
|---|---|
|방식|데이터 1비트씩 순차 전송 (TX, RX, GND 라인 등)|
|속도|일반적으로 9600 ~ 115200 bps 사용|
|전압|±12V 로 구분 (0/1)|
|연결|COM 포트 (USB to RS232 어댑터도 가능)|
|방향|양방향 가능 (Full Duplex)|

### 📦 코드에서

```c
OpenComConfig(...)   // 포트 열기
ComWrt(...)          // 데이터 쓰기
ComRd(...)           // 데이터 읽기
```

→ 이 모든 함수들이 RS232 기반이야.

---

## ✅ 2. MODBUS RTU란?

> **산업용 통신 프로토콜**로, 장비 제어에 널리 사용돼. RS232 위에 얹는 **데이터 형식 규칙**이라고 생각하면 돼.

### 🔗 RTU (Remote Terminal Unit) 모드

- 데이터는 **바이트 단위로 전송**
    
- 각 명령은 **주소, 명령, 데이터, CRC**로 구성
    

### 📑 구조 예시 (명령 하나)

|항목|예|설명|
|---|---|---|
|Slave ID|`0x01`|대상 장치 주소|
|Function|`0x03`|명령 종류 (ex. 레지스터 읽기)|
|Data|`0x0FCD, 0x0001`|시작주소 + 길이|
|CRC|`0xXXXX`|데이터 검증용|

### 📦 코드에서

```c
s_serial.send[0] = 0x01;     // 장치 ID
s_serial.send[1] = 0x10;     // 명령어 (0x10 = 여러 레지스터 쓰기)
...
AttachModbusCRC();          // CRC 붙임
```

---

## ✅ 3. CVI (LabWindows/CVI)

> National Instruments에서 만든 **C 언어 기반 GUI 프레임워크**야. 윈도우용 프로그램 만들 때 사용.

### 🖼️ 특징

|항목|설명|
|---|---|
|언어|C 기반|
|기능|UI 패널 만들기, 버튼/차트/텍스트박스 등 제공|
|장점|하드웨어 제어와 연동 쉬움 (DAQ, RS232 등)|

### 📦 코드에서

```c
DisplayPanel(control_handle);     // UI 패널 띄움
SetCtrlVal(control_handle, ID, value);  // UI에 값 표시
GetCtrlVal(control_handle, ID, &value); // 사용자 입력값 읽기
```

→ CVI 덕분에 버튼 누르면 펌프가 분사되는 식으로 구성할 수 있는 거야.

---

## ✅ 4. CRC (Cyclic Redundancy Check)

> **데이터가 손상 없이 도착했는지 검사하는 값**. 송신 시 계산해서 붙이고, 수신 후 다시 계산해 비교해.

### 🧮 예시

- "HELLO" 보내면 → `[HELLO][CRC16]`
    
- 수신 측에서 **HELLO의 CRC 다시 계산**
    
- 값이 다르면 → "데이터 손상!" 에러 발생
    

### 📦 코드에서

```c
AttachModbusCRC()         // 전송 전에 CRC 붙임
CheckModbusCRC()          // 수신 후 CRC 확인
```

---

## ✅ 5. SLIP 인코딩 (Serial Line Internet Protocol)

> **특수문자가 섞인 데이터를 안전하게 전송**하기 위해 인코딩하는 방식

### 💡 왜 필요할까?

- RS232 통신에서 **특수문자(C0, DB 등)**가 들어가면 데이터 끝으로 오해될 수 있음
    
- SLIP은 이를 피하기 위해 **데이터 바이트를 치환(escape)** 함
    

### 🔁 예시

|원래 바이트|SLIP 인코딩 결과|
|---|---|
|0xC0|0xDB 0xDC|
|0xDB|0xDB 0xDD|

### 📦 코드에서

```c
EncodeSLIP(s_serial.sendbyte, s_serial.send);
DecodeSLIP(s_serial.readbyte, s_serial.read);
```

→ 패킷이 잘려서 깨지는 것을 막기 위한 인코딩 과정이야.

---

## 📌 전체 흐름 예시로 정리하면

1. 사용자가 UI에서 "향수 분사" 누름 (CVI)
    
2. 선택값으로 MODBUS 명령 패킷 생성
    
3. SLIP 인코딩 + CRC 추가
    
4. RS232로 장치에 전송
    
5. 장치 응답 수신 → CRC 체크 → SLIP 디코딩
    
6. 결과 표시 (OK or 오류 메시지)
    

---

![[Pasted image 20250806105724.png]]