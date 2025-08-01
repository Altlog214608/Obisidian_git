네! 첨부하신 **ScentControl.cpp / ScentControl.h** 코드의 역할과 동작 원리를 쉽고 자세하게 단계별로 설명해 드리겠습니다.  
이해를 돕기 위해 함수별 목적과 동작 흐름, 변수 역할, 그리고 하드웨어 제어 관계 위주로 풀어서 설명합니다.

# 1. ScentControl.cpp / ScentControl.h 파일 개요

- **역할**:  
    디지털 향 시스템의 핵심 펌웨어 블록 중 하나로,  
    향 분사 및 세정 (Clear) 동작을 시간과 상태에 맞게 제어하는 **작업 루프와 명령 인터페이스**를 담당합니다.
    
- **주요 기능**:
    
    - 분사/세정 명령 수신 후, 타이밍 체크하며 솔레노이드 밸브를 켜고 끔 (향기 분사, 청소 모드)
        
    - 여러 향(솔레노이드 밸브)을 순차적으로 또는 특정 조건에 맞춰 운용
        
    - 현재 동작 상태를 저장 및 업데이트 (향 상태, 세정 상태)
        
    - 외부 명령 호출용 함수들 및 상태 조회 지원
        

# 2. 주요 전역 변수 설명

|변수명|의미 및 역할|
|---|---|
|`RunScentStatus`|현재 4개의 향 상태 상태값 ('0': OFF, '1': ON, '2' 등) 저장 (향 번호 1~4 대응)|
|`RunDelayTime`|상위 시스템(PC)에서 전달받은 발향 대기 시간 (delay, ms 단위 추정)|
|`RunPieriod`|발향 지속 시간 (period, 10ms 단위 추정)|
|`RunScentNo`|실행할 향 번호 (몇 번째 향을 분사할지 지정)|

|제어/루프 관련 변수|설명|
|---|---|
|`RunScentLoopFlag`|향 분사 루프 활성화 플래그 (1: 실행 중, 0: 중지)|
|`RunScentLoopIndex`|향 분사 상태 머신/루프 단계 인덱스|
|`RunClearLoopFlag`|세정 루프 활성화 플래그|
|`RunClearLoopIndex`|세정 루프 단계 인덱스|
|`_DoRunDelayTime`, `_DoRunPieriod`, `_DoRunScentNo`|처리용 내부 변수, 현재 실행 중인 delay, 기간, 향 번호|

# 3. 주요 함수 및 동작 원리

## 3-1. `SetScentStatus(int ScentNo, char Status)`

- 역할:  
    향 번호에 따라 `RunScentStatus` 배열 요소를 지정한 상태문자로 세팅함
    
- 향 번호 1~4를 배열 인덱스 0~3에 대응하여 해당 위치 값 변경 (예: '1' = 향 켜짐)
    

## 3-2. `RunScentLoop()` — 향 분사 루프 상태 머신

- 역할:  
    향 분사를 타임라인에 따라 단계별로 수행하는 상태 머신 함수
    
- 동작 단계 스위치(index):
    

|단계|동작 설명|
|---|---|
|START + 0|`ResetSol()`으로 모든 솔레노이드 OFF → `ReadyScentSol(_DoRunScentNo)` 호출 → 발향 준비 상태로 변경 (`_DoRunScentNo`에 맞는 밸브 조작)|
|START + 1|딜레이(`_DoRunDelayTime`) 동안 대기 (특정 시간 지난 후 다음 단계 진행)|
|RUN + 0|발향 시점 시작 → 타임스탬프 갱신, 상태 전환|
|RUN + 1|발향 지속 시간 추적 (`_DoRunPieriod`) → 완료되면 다음 단계 진행|
|END + 0|`ResetSol()` 하여 솔레노이드 끄고, 발향 상태 '0'(OFF)로 변경 후 종료|

- 호출에 따라 단계별 타이밍을 체크하며 솔레노이드(향 분사 밸브)를 켜고 끔
    

## 3-3. `DoScentBackLoop()`

- 상위에서 발향 명령 받았을 때 호출
    
- 내부 변수들(`_DoRunDelayTime`, `_DoRunPieriod`, `_DoRunScentNo`)에 현재 수행 파라미터 복사
    
- 발향 플래그(`RunScentLoopFlag`), 상태 인덱스 초기화
    
- `SetScentStatus`로 향 상태를 ‘1’ (발향 중) 으로 세팅
    
- 세정 루프가 실행 중이면 초기화
    
- (실행 완료 후 루프 진행은 메인 루프에서 `RunScentLoop()` 반복 호출로 처리)
    

## 3-4. `RunClearLoop()` — 세정 루프 상태 머신

- 발향 루프와 매우 유사하게 동작하지만 세정 밸브 제어 중심
    
- 단계 운영 방식도 같음: 시작(밸브 셋팅) → 딜레이 → 운전 시간 → 종료
    
- 세정에 맞게 `_DoRunClearNo`, `_DoRunDelayTime`, `_DoRunPieriod`라는 변수 사용
    
- 끝나면 밸브 끄고 `SetScentStatus`에 ‘0’으로 상태 변경
    

## 3-5. `DoClearBackLoop()`

- 세정 명령 시 호출
    
- 내부 변수에 세정 파라미터 세팅 후 세정 루프 및 인덱스 초기화
    
- 향 분사 루프가 동작 중이면 종료 처리한다는 점이 특징
    

## 3-6. `DoStopBackLoop()`

- 발향 및 세정 모두 중지 명령
    
- 발향 상태를 모두 초기화(`ResetSol()`), 상태 표시를 ‘0’으로 세팅
    
- 상태 플래그들을 모두 종료 상태로 초기화하며 안전하게 모든 밸브를 끔
    

## 3-7. `DoSetSolZeroValveBroadcastBackLoop()` 및 `DoSetSolZeroValveBackLoop()`

- 솔레노이드 밸브 0번(기본 밸브)의 위치 보정(Zeroing) 관련 처리
    
- Broadcast는 여러 장비에 동시에 명령 송신된 경우 처럼 주소에 따라 상태 참조
    
- `RunSetSolZeroValueStatusBroadcast`, `RunSetSolZeroValueStatus` 배열 내 ‘1’인지 아닌지에 따라 해당 밸브 켜거나 끔
    
- 향 및 세정 루프가 돌고 있으면 루프 인덱스 및 플래그를 종료 시킴
    

# 4. 동작 흐름 시각화 및 이해

text

`상위 명령 → DoScentBackLoop() 호출 → 상태/시간 등 변수 설정 → RunScentLoopFlag 켜짐 ↓ 메인 (또는 ISR) 루프에서 아래 반복 호출: if (RunScentLoopFlag) RunScentLoop()     -> 단계별(START, RUN, END)로 솔레노이드 밸브 ON/OFF 및 딜레이 체크 진행 if (RunClearLoopFlag) RunClearLoop()     -> 유사하게 세정 밸브 제어 및 단계 진행 발향 및 세정 명령 및 상태 관리가 루프별로 분리되어 부드럽게 병렬 운영 가능`

# 5. 주요 연동 함수 및 변수 요약

|함수|역할 설명|
|---|---|
|`SetScentStatus`|향 번호별 상태 저장 및 표시|
|`RunScentLoop`|향 분사 단계별 상태 머신 처리|
|`DoScentBackLoop`|발향 명령 수신 시 초기 설정 및 루프 시작|
|`RunClearLoop`|세정 상태 머신 처리|
|`DoClearBackLoop`|세정 명령 초기 설정 및 루프 시작|
|`DoStopBackLoop`|향 / 세정 모두 즉시 정지|
|`DoSetSolZeroValveBroadcastBackLoop`|밸브 0 보정广播 및 처리|
|`DoSetSolZeroValveBackLoop`|밸브 0 보정 단일 장비 처리|

# 6. 정리 및 팁

- **시간 기반 상태 머신 구조**: `LoopIndex` 변수로 단계 구분 → `millis()`로 경과시간 판단 → 다음 단계 전환
    
- **향 번호와 상태를 별도로 관리**: 상태 문자를 통해 장비 상태 추적 용이
    
- **발향과 세정을 독립된 루프로 병렬 처리** → 실제 하드웨어 동시 관리 가능
    
- **큰 작업 실행 함수(DoScentBackLoop 등)는 초기화 및 플래그 세팅 위주, 실제 수행은 RunScentLoop/RunClearLoop에서 단계적으로**
    
- **솔레노이드 밸브 제어는 `ResetSol()`과 `ReadyScentSol()`, `ReadyClearSol()`(IOControl.cpp 내) 함수와 연결되어 하드웨어 핀 제어가 이뤄짐**
    

필요하시면 발향/세정 루프 내에서 실제 솔레노이드 켜고 끄는 부분(예: `ReadyScentSol` 세부 동작이나 타이밍 제어 흐름)도 더 자세히 도와드릴 수 있습니다.

또한, 이 코드가 MCU 내에서 일반적으로 어떻게 메인 루프 안 또는 작업 스케줄러와 연동되어 실행되는지도 설명 가능하니, 언제든 요청해 주세요!