---
aliases:
  - 16_SW<->HW간
---
SW(PySide6 Python)에서 HW(펌웨어, C/C++)로 **바이트 패킷을 보내고, MCU(하드웨어)가 이를 복호화해서 어떤 향을 발향할지 판단하는 과정**을 단계별로 아주 쉽게 설명해 드릴게요.

# 1. PC 소프트웨어(파이썬)에서 바이트 패킷 만드는 과정

1. **검사 문항 등 UI에서 사용자의 선택/명령(향 번호, 펌프 세기 등)이 들어옴**
    
    - 예를 들어, 3번 향/60% 세기/10초 분사 등을 선택
        
2. **dsComm 등 내부 모듈에서 “향 번호, 명령 코드, 세기, 시간 등”을 바이트 단위로 패킹**
    
    - 파이썬의 struct.pack, or 리스트로 **`wdata`**라는 바이트 시퀀스 생성
        
    - 예시 wdata: `[STX][ID][명령][향번호][세기][시간][CRC16][ETX]`
        
        - 각 데이터는 1~2바이트(주로 아스키/바이너리)가 차지
            
3. **write_data(wdata)** 함수에서 실제로 포트로 전송
    
    - `self._serial.write(wdata)`
        
    - → 실제 하드웨어/MCU로 데이터가 전송됨
        

# 2. MCU(펌웨어)에서 패킷 복호화 과정

1. **UART 수신 인터럽트(ISR(USART0_RX_vect))에서 데이터 한 바이트씩 수신해서 버퍼에 저장**
    
2. **SerialHostEvent 등에서 STX(0x02)~ETX(0x03) 패킷 단위로 완성될 때까지 수집**
    
3. **RunSerialHostDataCheck 함수에서 패킷 명령 분리**
    
    - 명령코드가 R(분사)인지, L(세정)인지, G(상태)인지 확인
        
    - **향 번호/세기/시간** 등 파라미터를 아스키->숫자로 변환(예: ‘05’ → 5번 향)
        
4. **RunSerialHostCommandRunScent (발향), RunSerialHostCommandRunClear(세정)** 등
    
    - 받은 파라미터(향번호, 분사시간 등)를 내부 상태 변수에 저장
        

# 3. 어떤 향을 발향할지는 어디에서 결정되는가?

- **패킷에 “향 번호(예: ‘03’ 또는 0x03)”가 명확히 들어있음**
    
- MCU의 명령 처리부(`RunSerialHostCommandRunScent` 등)에서
    
    c
    
    `memcpy(Temp, &SerialHostRxCommandParam[0+5], 2);   RunScentNo = atoi(Temp);  // 향 번호 파싱(예: '05'→5)`
    
    → **이 값을 기준으로 하드웨어가 해당 향 채널의 솔레노이드 밸브만 ON**
    
- **향 채널 별 분사 로직**(ReadyScentSol 같은 함수로 밸브/펌프 조합 제어)
    
    - ex) ReadyScentSol(5)를 부르면, 5번 포트 밸브에만 전기 신호를 줘서 그 향만 분사
        

# 4. 전체 과정 요약(한 번 더 쉽게!)

1. **PC가 “향 3번!”이라는 내용을 바이트로 만들어 시리얼로 보냄**
    
2. **MCU(펌웨어)는 그 바이트(예: ‘03’)를 잘라 읽어 향 번호 변수에 저장**
    
3. **그 번호에 매칭된 하드웨어 밸브만 열어서 해당 향만 분사**
    
    - 실제 어느 밸브를 열지는 IOControl.cpp의 SolOn/ReadyScentSol 등에서 핀번호로 결정!
        
4. **분사 끝나면 상태를 다시 패킷으로 만들어 PC로 응답**
    
    - (“분사 끝났다”, “상태 OK” 등)
        

# 5. 보충: 데이터 복호화(파싱) 핵심 코드

(HostRun.cpp 등에서 발췌·의역)

c

`memcpy(Temp, &SerialHostRxCommandParam[파라미터_오프셋], 2); // 향번호 두 자리 RunScentNo = atoi(Temp);  // '03' → 3번 향 ReadyScentSol(RunScentNo);  // 해당 향 준비 및 분사시작`

# 6. 결론:

**향을 정하는 정보는 바이트 패킷(특히 “향 번호”)에 있고, 펌웨어가 이 값을 복호화해서 하드웨어의 대응되는 밸브(채널)만 열어 그 향만 분사한다!**

—

더 궁금한 부분(예: 실제 향-밸브 매칭 테이블, 딜레이/시간 적용법, 세정/정지 분기, 에러 핸들링 등)도 추가로 설명 가능합니다.  
원하시는 항목 말씀해 주세요!

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d454c19b-06fb-4a6d-9714-a94ec292d75d/ScentSmart.py](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d454c19b-06fb-4a6d-9714-a94ec292d75d/ScentSmart.py)