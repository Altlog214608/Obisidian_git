아래는 주신 `NFSerial` C++ 시리얼 통신 클래스(`nfserial.h`와 `nfserial.cpp`)를 **아주 쉽게, 자세히, 단계별로** 풀어 설명한 내용입니다.  
어렵고 전문적인 MCU/AVR 레지스터 직접 제어 부분도 최대한 비유와 함께 초심자가 이해할 수 있도록 정리했습니다.

# NFSerial 클래스 개요

- **목적**  
    AVR MCU(또는 호환 마이크로컨트롤러)에서 UART(시리얼) 통신을 직접 제어하고,  
    버퍼링 및 인터럽트 처리까지 포함한 **시리얼 입출력 라이브러리** 역할을 하는 클래스입니다.
    
- **이 클래스는** Arduino의 `HardwareSerial`과 비슷한 기능을 하지만,  
    UART 하드웨어 레지스터를 직접 포인터로 받아 조작합니다.
    
- **하드웨어** :  
    MCU UART 레지스터들 (UBRRnH/L, UCSRnA/B/C, UDRn) 포인터를 전달 받아 사용
    

# 1. 주요 멤버 변수 및 역할

|변수명|설명|
|---|---|
|`_ubrrh, _ubrrl`|Baud rate 설정용 하드웨어 레지스터 포인터 (전송속도 설정)|
|`_ucsra, _ucsrb, _ucsrc`|시리얼 상태 및 제어 레지스터 포인터|
|`_udr`|실제 송수신 데이터 레지스터 포인터|
|`_rx_buffer`|수신 버퍼 (고정 크기 배열, 64바이트)|
|`_tx_buffer`|송신 버퍼 (고정 크기 배열, 64바이트)|
|`_rx_buffer_head/tail`|수신 버퍼에서 다음에 쓸 위치와 읽을 위치|
|`_tx_buffer_head/tail`|송신 버퍼에서 다음에 쓸 위치와 읽을 위치|
|`_written`|쓰기 작업 발생 여부 플래그|
|`_TxEndFlag`|송신 종료 플래그|

- 버퍼 관련 변수는 FIFO(선입선출 큐)로 동작하여, 수신·송신 버퍼를 순환 관리
    
- 레지스터 포인터를 통해 직접 MCU 레지스터를 읽고 씀
    

# 2. 생성자 & 초기화

cpp

`NFSerial::NFSerial(     volatile uint8_t* ubrrh, volatile uint8_t* ubrrl,    volatile uint8_t* ucsra, volatile uint8_t* ucsrb,    volatile uint8_t* ucsrc, volatile uint8_t* udr) :    _ubrrh(ubrrh), _ubrrl(ubrrl),    _ucsra(ucsra), _ucsrb(ucsrb),    _ucsrc(ucsrc), _udr(udr),    _rx_buffer_head(0), _rx_buffer_tail(0),    _tx_buffer_head(0), _tx_buffer_tail(0) { }`

- MCU UART 하드웨어 레지스터가 저장된 메모리 주소를 포인터로 받아서 초기화
    
- 수신, 송신 버퍼 위치 초기화
    
- 외부에서 특정 UART 포트의 하드웨어 레지스터 주소를 넘겨줘야 작동
    

# 3. UART 초기화 (`begin`)

cpp

`void NFSerial::begin(unsigned long baud, byte config)`

- **baud** : 전송 속도 (ex: 9600, 115200)
    
- **config** : 데이터 비트, 패리티, 스톱비트 설정 (예: SERIAL_8N1)
    
- 내부적으로 **baud_rate 레지스터(UBRR)** 계산 및 설정
    
- U2X (2배 속도) 모드 자동 시도 및 부득이시 기본 모드로 전환
    
- 데이터 프레임 설정 및 UART 송수신/인터럽트(enable) 설정
    

# 4. 데이터 수신 처리 - 인터럽트 핸들러 `_rx_complete_irq`

cpp

`void NFSerial::_rx_complete_irq(uint8_t c) {     rx_buffer_index_t i = (unsigned int)(_rx_buffer_head + 1) % NF_SERIAL_RX_BUFFER_SIZE;    if (i != _rx_buffer_tail)  // 수신 버퍼가 꽉 차지 않았다면    {        _rx_buffer[_rx_buffer_head] = c;  // 받은 바이트 저장        _rx_buffer_head = i;               // 버퍼 포인터 업데이트    } }`

- MCU의 RX 인터럽트가 발생하면 호출되어, 수신된 바이트(`c`)를 버퍼에 저장
    
- 버퍼가 가득 찬 경우 덮어쓰지 않고 무시(데이터 손실 방지)
    
- 수신버퍼는 원형 버퍼(RingBuffer) 형태
    

# 5. 데이터 송신 처리 - 인터럽트 핸들러 `_tx_udr_empty_irq`

cpp

`void NFSerial::_tx_udr_empty_irq(void) {     unsigned char c = _tx_buffer[_tx_buffer_tail];  // 송신할 다음 바이트    _tx_buffer_tail = (_tx_buffer_tail + 1) % NF_SERIAL_TX_BUFFER_SIZE; // 버퍼 포인터 이동     *_udr = c;   // 실제 UART 데이터 레지스터에 씀 (즉시 전송 시작)     // TX 완료 플래그 클리어 (하드웨어 레지스터 조작)    // TXC0 비트 1로 설정 해제: 이러면 flush() 함수가 바이트 전송 완료를 기다릴 수 있음 #ifdef MPCM0     * _ucsra = ((*_ucsra) & ((1 << U2X0) | (1 << MPCM0))) | (1 << TXC0); #else     * _ucsra = ((*_ucsra) & ((1 << U2X0) | (1 << TXC0))); #endif }`

- 송신 버퍼에서 다음 바이트 꺼내서 UART 데이터 레지스터에 쓰며 전송시작
    
- TX 완료 플래그(TXC0) 클리어 처리
    
- 버퍼 순환관리를 통해 연속 송신 지원
    

# 6. 읽기/쓰기/기본 API 함수 소개

|함수|설명|
|---|---|
|`available()`|수신 버퍼 내 읽을 수 있는 바이트 수 리턴|
|`read()`|수신 버퍼에서 다음 바이트를 읽어 반환 (없으면 -1)|
|`peek()`|다음 읽을 바이트를 버퍼에서 미리 확인 (읽기 없이)|
|`write(uint8_t c)`|송신 버퍼에 1바이트 쓰기, 바로 보내기 가능 시 즉시 레지스터에 쓰기도 함|
|`flush()`|송신 버퍼가 비워지고 실제 모든 데이터가 다 전송될 때까지 대기|
|`end()`|송수신 중지, 인터럽트 비활성화 및 버퍼 초기화|
|`availableForWrite()`|송신 버퍼에 쓸 수 있는 공간 크기 리턴|

# 7. 송신 함수 `write()` 상세 설명

- 만약 송신 버퍼가 비어 있고 UART 데이터 레지스터가 전송 준비 완료 상태이면,  
    단순히 레지스터에 바로 쓴다 → 전송 지연 최소화
    
- 그렇지 않을 경우 -> 송신 버퍼에 저장
    
- 송신 버퍼가 가득 찼으면, 인터럽트 핸들러가 빈 공간 만들 때까지 대기(or 인터럽트 활성화 후 핸들러에 맡김)
    
- 버퍼 내부 포인터 원자적 처리(락)으로 다중 인터럽트 상황 방지
    

# 8. 기타 중요 코드 요소

- `reset_rx_queue()`: 수신 버퍼 초기화
    
- _head, _tail 버퍼 인덱스 관리로 버퍼의 원형 큐 기능 지원
    
- ATOMIC_BLOCK, `sbi()`, `cbi()` : 비트 단위 레지스터 제어 및 인터럽트 락 구현 (AVR 저수준 제어)
    

# 9. 함수 동작 비유

|기능|비유 설명|
|---|---|
|`_rx_complete_irq()`|편지함에 새 편지가 도착하면 놓음|
|`read()`|편지함에서 다음 편지 하나 꺼냄|
|`_tx_udr_empty_irq()`|메일박스가 비면 다음 보내야 할 편지 넣음|
|`write()`|보낼 편지를 우체통에 차곡차곡 쌓음|
|`flush()`|모든 편지가 발송 완료될 때까지 기다림|
|`begin()`|우체국 문 열고 업무 시작(통신 속도 설정 등)|
|`end()`|우체국 문 닫고 업무 종료|

# 10. 요약

- `NFSerial`은 MCU 하드웨어 UART 레지스터를 직접 제어하는 완전 저수준 시리얼 통신 라이브러리
    
- 수신·송신 버퍼를 원형 큐로 관리해 데이터 흐름 매끄럽게 처리
    
- ISR 기반 인터럽트 처리로 UART 이벤트(수신, 송신 버퍼 빈 상태)를 빠르게 반응
    
- Arduino 같은 플러그인 없이도 직접 포인터 주입으로 범용성 높은 통신 가능
    
- PySide6 기반 PC SW `write_data` 함수와 연결해 하드웨어와 양방향 통신 구현에 핵심적
    

필요하다면 ISR, 비트 연산, 버퍼 동기화, 인터럽트 우선순위 등 임베디드 세부 기법,  
또는 이 라이브러리와 PC 소프트웨어 간 실제 데이터 주고받는 예시, 기타 함수 활용법 등도 추가로 상세 설명 가능합니다. 알려주세요!