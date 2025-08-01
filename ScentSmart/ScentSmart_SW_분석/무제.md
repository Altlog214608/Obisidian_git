파일 첨부 확인했습니다!  
이전까지 설명드린 ScentSmart, dsComm, dsSerial 등 전체 하드웨어-소프트웨어 연결, 패킷 구조, 함수 동작 방식에 근거해서 아래와 같이 한층 더 확실하게 정리드립니다.

# 1. **convertLevelToPumpPower() 함수 위치와 역할**

첨부된 **dsComm.py** 파일에서 바로 정의되어 있습니다.

```python

def convertLevelToPumpPower(value):

    if value == 1:

        return 17

    elif value == 2:

        return 40

    elif value == 3:

        return 85

    else:

        return 40

```

- **역할 :**  
    소프트웨어 상의 '파워 레벨(1, 2, 3)' 숫자를 하드웨어(분사 펌프)가 이해하는 값(17, 40, 85) 으로 변환합니다.
    
    - **사용처 :**
        
        - `sendMsgForEmitClean()`, `sendMsgForClean()` 등에서 각 파워별로 사용.
            
- **예시 :**
    
    - UI/세팅에서 펌프 파워 "1단계"를 선택하면 → 하드웨어 패킷 내 값 `17`로.
        
    - "2단계" → 40, "3단계" → 85.
        
    - 범위 밖의 값이 들어오면 40으로 고정(에러 방지용 default).
        

# 2. **crc16_modbus() 함수**

**dsComm.py** 파일에 정의되어 있으며, **패킷 데이터 무결성 검사(체크섬)용**입니다.

- **동작원리 :**
    
    - 입력 데이터(패킷 바이트들)에 대해 바이트별로 특정 규칙(CRC 테이블)으로 2바이트(16비트)의 결과값을 산출.
        
    - 이 값을 패킷 끝에 붙이면 하드웨어/소프트웨어가 통신 중 손상이나 변조를 즉시 감지 가능.
        
- **핵심 코드 (부분 발췌):**
    
    ```python

def crc16_modbus(init_crc, dat, len):

    crc = [init_crc >> 8, init_crc & 0xFF]

    for b in dat:

        tmp = crc16_table[crc[0] ^ b]

        crc[0] = (tmp & 0xFF) ^ crc[1]

        crc[1] = tmp >> 8

    return struct.pack("H", address)  # 실제에선 CRC 2바이트 반환

```
    
    (패킷 만드는 부분에서 실제로 mdata에 붙여 전송)
    
- **사용처:**  
    sendMsgForEmitClean(), sendMsgForClean(), sendMsgWriteSingleRegister(), sendMsgReadRegister() 등 바이트 패킷 생성 후 항상 마지막에 붙임.
    

# 3. **시리얼 포트 연결 구조 (`dsSerial.py`)**

- **포트 연결:**
    
    - PySerial 기반(혹은 Qt 내장)으로 물리 포트를 엽니다.
        
    - 주요 함수:
        
        - `_connect()`/`_connect_default()` : 포트, 보드레이트 등 설정 후 연결.
            
        - `_is_open()` : 현재 포트 열림 상태 체크.
            
        - `_disconnect()` : 포트 연결 해제.
            
- **시리얼 쓰레드:**
    
    - `SerialReadThread` : 포트로 들어오는 데이터를 실시간 읽어서 Qt 시그널로 메인프로그램(UI)에 돌려줍니다(비동기 수신 구조).
        
    - 데이터가 들어오면 `_serial_received_data.emit(buf)`로 이벤트 전달.
        

# 4. **dsComm 프로토콜 설계와 하드웨어 대응 구조**

- **통신 포맷(패킷 구조)**
    
    - [ID][FUNC][ADDRESS][QOR][LENGTH][CMD][SCENT_NO][SCENT_PUMP_POWER][CLEAN_PUMP_POWER][SCENT_PERIOD][CLEAN_PERIOD][SCENT_DELAY][CLEAN_DELAY][CRC16]
        
    - 각 항목은 1~2바이트로 빅엔디언(상위바이트 우선) 포장됨.
        
    - `sendMsgForEmitClean()` 등에서 상세히 조합.
        
- **프로토콜 특성**
    
    - **ID**: 장치 고유값(여러 장치 쓰는 경우 구분)
        
    - **FUNC**: 명령 코드(0x10 = 다중레지스터쓰기 등)
        
    - **CMD**: 실제 동작(발향, 세정, 중지 등)
        
    - 나머지는 향 번호, 세기, 시간 등 실제 동작 제어값
        
- **하드웨어 대응**
    
    - 소프트웨어(UI 또는 내부 로직)에서 사용자의 "향·세기·시간" 선택값을 실수값 → 변환함수로 바이트 패킷 완성 → 패킷 전송
        
    - 하드웨어가 패킷을 그대로 해석해 실동작(분사, 세정 등)
        
    - 응답 시 상태/이벤트 코드가 같은 방식 패킷으로 들어옴 → 파싱 후 콘솔/로그/UI에 반영
        

---
##  `write_data()` 함수 (ScentSmart 클래스 내부)

  

```python

def write_data(self, wdata):

    if dsSerial._is_open(self._serial):

        self._serial.write(wdata)

        self._serial_console.append("TX(%d):" % len(wdata) + str(wdata.hex()))

        self._serial_console.moveCursor(QtGui.QTextCursor.End)

        print("TX(%d):" % len(wdata) + str(wdata.hex()))

    else:

        self._serial_console.append(dsText.serialText['status_close'])

        self._serial_console.moveCursor(QtGui.QTextCursor.End)

        print(dsText.serialText['status_close'])

        self.uiDlgMsgText(dsText.errorText['disconnect'])

```

  

- **역할:** 바이트 데이터를 시리얼 포트로 전송 + 로그 출력

  

---


## `requestScentNo()` 함수

  

```python

def requestScentNo(self, scent_no, command):

    sendMsg = dsComm.sendMsgForEmitClean(

        id=1,

        func=16,

        address=dsComm.ADDRESS_RUN_CLEAR,

        qor=8,

        data_length=16,

        data_command=command,

        data_scent_no=scent_no,

        data_scent_pump_power=dsSetting.dsParam['scent_power'],

        data_clean_pump_power=dsSetting.dsParam['cleaning_power'],

        data_scent_period=dsSetting.dsParam['scent_run_time'],

        data_clean_period=dsSetting.dsParam['cleaning_run_time'],

        data_scent_delay=dsSetting.dsParam['scent_post_delay'],

        data_cleanup_delay=dsSetting.dsParam['cleaning_post_delay']

    )

    self.write_data(sendMsg)

```

  
- **역할:** 향 번호와 명령 기반으로 발향/세정 패킷 생성 후 시리얼 전송

  

---
# 5. **전체 동작 시퀀스**

1. **사용자 UI에서 발향 등 명령 선택**
    
2. → 파라미터가 `requestScentNo()`로 들어감
    
3. → `sendMsgForEmitClean()`에서 모든 파라미터를 변환·포장해 바이트 시퀀스 생성
    
4. → `write_data()`에서 시리얼 포트 통해 패킷 송신
    
5. → 하드웨어가 받은 패킷 해석, 명령 실행
    
6. → 필요 시 하드웨어에서 응답 패킷 전송 → 소프트웨어에서 파싱해 실시간 상태/로그/에러 UI에 반영
    

# **핵심 요약/비유**

- **convertLevelToPumpPower:**  
    사용자의 "세기 1~3단"을 하드웨어가 먹는 실수(예: 17, 40, 85)로 바꿔주는 "변환 어댑터" 역할.
    
- **crc16_modbus:**  
    패킷을 전송 전/후로 "시그니처(지문)" 붙여, 통신이 꼬이면 바로 알아차리게 하는 안전장치.
    
- **dsSerial 시리얼 쓰레드:**  
    신호등 지키는 경찰처럼, 포트 입출력을 따로 실시간으로 관리.
    
- **dsComm 패킷 구조:**  
    꼼꼼히 규정된 주소·명령·데이터 포맷 = 하드웨어-소프트웨어 양방향 통역사!
    
- **전체 대응:**  
    SW UI/로직 → (값 변환) → 바이트 패킷 생성 → 시리얼 전송 → HW 제어 → (상태패킷 수신) → SW 상태업데이트
    

