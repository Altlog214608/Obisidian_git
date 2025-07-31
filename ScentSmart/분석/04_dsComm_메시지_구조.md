# dsComm 메시지 구조 및 CRC 처리
tags: #CRC #MODBUS #시리얼메시지

## 📦 주요 메시지 생성 함수

- `sendMsgForEmitClean(...)`
- `sendMsgWriteSingleRegister(...)`
- `sendMsgReadRegister(...)`

## 🧪 CRC16 구조

- CRC 계산용 테이블 사용 (`crc16_table`)
- `crc16_modbus()` 함수로 패킷 말미에 체크섬 추가

## 💡 MODBUS 구조

```
[ID][FUNC][ADDR][QOR][DATA][CRC]
```

- Function Code: 6 (단일 쓰기), 16 (다중 쓰기), 4 (읽기)
