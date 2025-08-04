  

# AVR 기반 향 분사 시스템 - Modbus/펌프 모듈 분석 (Obsidian 문서)

  

이 문서는 `ConfigCOM`, `ControlPumpNF`, `FunctionSerial`, `MODBUS` 모듈에 대한 전체 소스코드 설명을 포함합니다.  

핵심 주제는 **MODBUS 통신을 이용한 향기 분사 제어**입니다.

  

---

  

## 🔧 1. ConfigCOM: 포트 설정 및 시리얼 구조체

  

```c

#define SIZEBUFFER 512

  

struct s_SERIAL {

  unsigned char read[SIZEBUFFER], send[SIZEBUFFER];

  int sendbyte, readbyte;

  int port, baudrate, parity, databits, stopbits, error;

  double timeout;

  char flagopen;

  char starter, terminator;

  int protocol, crc;

  int mode, debug;

  int add;

};

  

struct s_SERIAL s_serial;

```

  

- 포트 설정 및 입출력 버퍼 구성

- SLIP/PPP/CRC/MODBUS 방식 처리용 설정 포함

  

```c

void ReadConfigCom(struct s_SERIAL *scom);

int ActivateCom(struct s_SERIAL *scom);  

int ReadFromCom(struct s_SERIAL *scom);

int ReadSLIPCom(struct s_SERIAL *scom);

int SendToCom(struct s_SERIAL *scom);

void ResetCom(struct s_SERIAL *scom);

void ErrorCom(struct s_SERIAL *scom);

void QuitCom(struct s_SERIAL *scom);

```

  

---

  

## 💡 2. ControlPumpNF: 향 분사/세정 기능 정의

  

```c

#define NF_CMD 0x1068  // 명령: 1-run, 2-clean, 3-stop, 4-run&clean

#define NF_BNO 0x1069  // 향 번호

#define NF_RPW 0x106A  // run pump power

#define NF_CPW 0x106B  // clean pump power

...

#define NF_TOD 0x1073  // Turn off the device

```

  

### 구조체

  

```c

typedef struct {

  int cmdid, scentno, power, level, runtime;

} S_SCENT;

  

S_SCENT s_scent;

```

  

### 제어 함수

  

```c

void InitProcess(void);

void DecodePacket(void);

void WriteSingleRes_NF(unsigned short add, unsigned short value);

void WriteMultiRes_NF(unsigned short add, unsigned short cmdvalue);

void ReadRegisters_NF(unsigned char readcmd, unsigned short add, unsigned short qtres);

```

  

---

  

## 🔁 3. FunctionSerial: SLIP/MODBUS CRC 처리

  

### SLIP 인코딩/디코딩

  

```c

unsigned int DecodeSLIP(unsigned int nbytes, unsigned char *string);

unsigned int EncodeSLIP(unsigned int nbytes, unsigned char *string);

```

  

### CRC 처리

  

```c

char CheckCRC16(unsigned int nbytes, unsigned char *string);

void AttachCRC16(unsigned int nbytes, unsigned char *string);

char CheckModbusCRC(unsigned int nbytes, unsigned char *string);

void AttachModbusCRC(unsigned int nbytes, unsigned char *string);

```

  

SLIP 전송 시 C0, DB 특수 바이트 인코딩 처리 포함  

MODBUS용 CRC16 체크 및 추가 포함

  

---

  

## 🔄 4. MODBUS: 실제 명령 생성

  

### 명령 상수

  

```c

#define MBC_RHR 0x03 // Read Holding Register

#define MBC_RIP 0x04 // Read Input Register

#define MBC_WSR 0x06 // Write Single Register

#define MBC_WMR 0x10 // Write Multiple Registers

```

  

### 사용 예

  

```c

void WriteSingeRes_MODBUS(unsigned short add, unsigned short value);

void ReadRegisters_MODBUS(unsigned char readcmd, unsigned short add, unsigned short qtres);

```

  

→ s_serial.send 버퍼에 직접 명령 조립 후 `SendToCom()` 호출

  

---

  

## 📦 통신 흐름 요약

  

1. `InitProcess()`에서 시리얼 설정 + 포트 오픈

2. 명령 호출 (NF_CMD, BNO 등 주소지정)

3. `WriteSingeRes_MODBUS()` → MODBUS 명령 구성

4. `AttachModbusCRC()`로 CRC 추가

5. `SendToCom()`을 통해 전송

6. 펌프 제어 또는 상태 읽기 수행

  

---

  

## 🔚 기타 설정 및 주석 정보

  

- 펌프 방식: Kamoer(PWM) vs Thomas(Voltage)

- `NF_CTP` 주소로 타입 지정

- `proj_dir`, `modbus_cmd`, `wait_bytes` 등은 전역 통신 설정

  

---