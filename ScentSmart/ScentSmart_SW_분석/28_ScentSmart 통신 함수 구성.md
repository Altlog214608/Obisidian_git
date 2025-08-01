  

# ScentSmart 통신 함수 구성 정리 (requestScentNo, CRC, 포트 연결 등)

  

## 1. `convertLevelToPumpPower()` 함수는 어디에 있는가?

  

이 함수는 **펌프 세기(레벨)** 값을 하드웨어가 이해하는 **펌프 파워 값(주로 PWM 신호값)**으로 변환하는 용도입니다.

ScentSmart.py에는 정의가 없기 때문에, `dsComm.py` 또는 `dsSetting.py` 같은 **외부 모듈**에서 정의되어 있습니다.

  

```python

def convertLevelToPumpPower(level):

    # 예시: 1~10 레벨을 0~255 PWM으로 변환

    return int(level * 25.5)

```

  

---

  

## 2. `crc16_modbus()` 함수

  

**Modbus 통신 규약**에 따라 바이트 스트림의 **무결성 체크**를 위한 CRC16 체크섬을 생성합니다.

  

```python

def crc16_modbus(crc, data, length):

    crc = 0xFFFF

    for pos in range(length):

        crc ^= data[pos]

        for i in range(8):

            if crc & 0x0001:

                crc >>= 1

                crc ^= 0xA001

            else:

                crc >>= 1

    return crc.to_bytes(2, 'little')

```

  

---

  

## 3. 시리얼 포트 연결 (`dsSerial` 모듈)

  

PySerial 기반 시리얼 통신 객체를 사용합니다:

  

```python

import serial

  

ser = serial.Serial(

    port='COM3',

    baudrate=9600,

    bytesize=8,

    parity='N',

    stopbits=1,

    timeout=1

)

```

  

ScentSmart에서는 보통 `_serial` 속성으로 내부 저장, `_connect`, `_is_open`, `write_data()` 등 함수와 연계됩니다.

  

---

  

## 4. dsComm 통신 프로토콜 구조

  

### 명령 프레임 구성 예시:

  

```

[ID][FUNC][ADDRESS][QOR][LEN][CMD][SCENT_NO][SCENT_PWR][SCENT_TIME][CLEAN_PWR][CLEAN_TIME][SCENT_DELAY][CLEAN_DELAY][CRC16]

```

  

각 필드는 보통 1~2바이트이며, `struct.pack(">H", value)` 형태로 바이트 변환됩니다.

  

---

  

## 5. `write_data()` 함수 동작

  

```python

def write_data(self, wdata):

    if dsSerial._is_open(self._serial):

        self._serial.write(wdata)

        self._serial_console.append("TX(%d):"%len(wdata) + str(wdata.hex()))

        self._serial_console.moveCursor(QtGui.QTextCursor.End)

        print("TX(%d):"%len(wdata) + str(wdata.hex()))

    else:

        self._serial_console.append(dsText.serialText['status_close'])

        self._serial_console.moveCursor(QtGui.QTextCursor.End)

        print(dsText.serialText['status_close'])

        self.uiDlgMsgText(dsText.errorText['disconnect'])

```

  

---

  

## 6. 전체 동작 흐름 요약

  

```plaintext

사용자 입력

    ↓

requestScentNo 호출

    ↓

sendMsgForEmitClean → convertLevel → struct → crc16 → 바이트 패킷 생성

    ↓

write_data로 전송

    ↓

하드웨어 실행

    ↓

수신 응답 수신 및 로그 출력

```

  

---

  

## 참고 예시: 향 5번, 60% 세기, 8초 분사

  

1. `requestScentNo(5, CMD)`

2. 내부 세기 변환 → 153

3. `sendMsgForEmitClean` → 바이트 배열 조립

4. `crc16_modbus` → 체크섬 계산

5. `write_data`로 포트 전송

6. 하드웨어 명령 실행

  

---

  

**문의나 추가 설명이 필요한 구조가 있다면 언제든 확장해 드릴 수 있습니다.**