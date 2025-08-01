아래는 공유해주신 **TimerOne.cpp / TimerOne.h** 코드에 대한 쉽고 자세한 설명입니다.  
16bit 타이머를 이용한 PWM 및 인터럽트 기능을 구현한 라이브러리로, ATmega1280 등 AVR, Teensy 등 임베디드 MCU에서 주로 사용하는 코드입니다.

# TimerOne 클래스 개요

- **목적**: ATmega168, ATmega328, ATmega1280, Teensy 등 MCU에서 16비트 타이머1를 이용해
    
    - 정밀한 **PWM 제어**
        
    - 원하는 주기로 **인터럽트 발생**
        
    - 타이머 주기(period) 설정과 재시작 등을 편하게 할 수 있도록 도와주는 유틸리티 클라스입니다.
        
- **주요 기능**:
    
    - 타이머 초기화 및 주기 설정 (`initialize()`, `setPeriod()`)
        
    - 타이머 시작/정지/재시작 (`start()`, `stop()`, `restart()`)
        
    - PWM 출력 채널 제어 (`pwm()`, `setPwmDuty()`, `disablePwm()`)
        
    - 사용자 정의 인터럽트 핸들러 연결 (`attachInterrupt()`, `detachInterrupt()`)
        

# TimerOne.cpp 주요 부분 설명

## 1. ISR(Interrupt Service Routine; 인터럽트 서비스 루틴)

- MCU에 따라 타이머 인터럽트 벡터가 다름
    
- `TimerOne` 객체의 `isrCallback()` 함수를 호출하게 하여,  
    사용자 임의로 정의한 함수가 타이머 인터럽트 시 실행됩니다.
    
- AVR ATtiny85 예:
    
    cpp
    
    `ISR(TIMER1_COMPA_vect) {   Timer1.isrCallback(); }`
    
- AVR ATmega328 등:
    
    cpp
    
    `ISR(TIMER1_OVF_vect) {   Timer1.isrCallback(); }`
    
- Teensy MCU 계열도 각각 별도 ISR 구현
    
- `isrCallback`은 기본값이 `isrDefaultUnused()`로 초기화되어있고,  
    필요하면 사용자가 `attachInterrupt()`로 원하는 함수로 변경할 수 있습니다.
    

## 2. TimerOne 정적 멤버변수

cpp

`unsigned short TimerOne::pwmPeriod = 0; unsigned char TimerOne::clockSelectBits = 0; void (*TimerOne::isrCallback)() = TimerOne::isrDefaultUnused;`

- `pwmPeriod` : 타이머 카운트 최댓값 (주기 결정용)
    
- `clockSelectBits` : 클럭 분주기 설정값
    
- `isrCallback` : 타이머 인터럽트 시 호출할 사용자 함수 포인터
    

## 3. initialize() 함수 (예: ATtiny85)

- 타이머1을 Clear Timer on Compare (CTC) 모드로 설정 (타이머가 OCR1C 레지스터 값과 일치하면 리셋됨)
    
- OCR1A Compare Match 인터럽트 사용 설정
    
- 기본 주기로 타이머 초기화
    

## 4. setPeriod(unsigned long microseconds)

- **입력**: 주기(마이크로초 단위)
    
- **역할**: 타이머의 분주기와 카운트 값을 적절히 설정하여 주기 조절
    
- 내부 계산식:
    
    - `cycles = microseconds * ratio` (_ratio는 CPU 클럭 관련 상수_)
        
- cycles 값에 따라 **분주기 클럭 설정 비트(clockSelectBits)** 선택(1,2,4,8,... 16384 배까지)
    
- PWM 주기 값(`pwmPeriod`) 계산 후 OCR 비교값에 세팅
    

## 5. 시작, 정지, 재시작 함수들

- `start()`: 타이머 클럭을 정지한 후 카운터를 0으로 리셋, 다시 재개
    
- `stop()`: 타이머 동작 중지 (CTC 모드 유지)
    
- `restart()`: start() 호출과 동일
    
- `resume()`: stop() 후 분주비 재설정 및 타이머 재개
    

## 6. attachInterrupt(), detachInterrupt()

- `attachInterrupt(void(*isr)())`:
    
    - 사용자 지정 함수(`isr`)를 인터럽트 콜백으로 등록
        
    - 타이머 OCR1A Compare Match 인터럽트 활성화 (`TIMSK |= _BV(OCIE1A)`)
        
- `attachInterrupt(void(*isr)(), unsigned long microseconds)`
    
    - 원하는 인터럽트 주기 지정 후 attachInterrupt
        
- `detachInterrupt()`
    
    - 타이머 인터럽트 비활성화 (`TIMSK &= ~_BV(OCIE1A)`)
        

# TimerOne.h 주요 내용 요약

- C++ 클래스 `TimerOne` 선언부
    
- MCU별 타이머 해상도 정의 (ATtiny85: 8bit, 나머지 MCU는 16bit)
    
- 함수 선언 및 정적 멤버 선언
    
- 다양한 MCU 지원 조건 컴파일 구문으로 타이머 초기화, 설정, 제어 구현
    

# 동작 원리 쉽게 비유하기

|개념/기능|비유 설명|
|---|---|
|타이머 카운터|시계 초침 같은 카운터, 특정 값(OCR1A)에 도달하면 시계 다시 돌아감|
|분주기 클럭|시계 초침 속도 조절 다이얼 (빠르게, 느리게)|
|인터럽트 서비스 루틴|알람이 울렸을 때 수행할 작업을 미리 예약해두는 것|
|PWM 출력을 위한 OCR|조명 밝기 조절용 스위치가 ON/OFF 되는 타이밍 결정|
|attachInterrupt()|“알람 울리면 이 함수 실행하세요” 라고 예약|
|stop() / start()|시계 멈추기 / 다시 작동시키기|

# 정리

- `TimerOne`은 16bit 타이머를 쉽게 쓸 수 있도록 만든 라이브러리입니다.
    
- 타이머의 클럭 설정과 주기 변경을 자동으로 조절해서 안정적인 PWM 및 인터럽트를 지원합니다.
    
- 사용자는 인터럽트 콜백 함수 연결, PWM 출력 설정, 타이머 시작/멈춤만 신경 쓰면 됩니다.
    
- 다양한 MCU를 지원하기 때문에 조건부 컴파일로 MCU별 맞춤 설정을 포함하고 있습니다.
    

# 참고 사항

- **ATtiny85는 타이머1이 8bit로 제한적**이므로 일부 기능(특히 고해상도 PWM)은 미지원
    
- Teensy나 ARM MCU에서는 상이한 레지스터 및 방식 적용, 완전히 다른 인터럽트 처리 함수 제공
    
- 이런 라이브러리는 Arduino 프로젝트 또는 MCU 임베디드 프로그래밍에서 하드웨어 타이머를 직접 다루기 어려울 때 큰 도움을 줍니다.
    

궁금하신 구체 함수 사용법, PWM 실제 적용 예제, 인터럽트 활용 팁, 그리고 MCU별 차이나 하드웨어 상세 설명 원하시면 추가 안내해 드릴 수 있습니다!

1. [http://labs.ideo.com](http://labs.ideo.com/)