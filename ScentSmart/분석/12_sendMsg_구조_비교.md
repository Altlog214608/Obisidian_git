# ✅ dsComm 시리얼 명령 함수 파라미터와 구조 비교

## 🔹 1. `id`, `func`, `address`, `qor`의 의미

| 필드     | 설명 |
|--------|------|
| `id`   | 장치 식별 번호. 동일 버스 내 여러 장치를 구별 |
| `func` | 명령의 종류(Function code). 예: 6(단일 쓰기), 16(다중 쓰기) |
| `address` | 명령이 적용될 시작 주소(레지스터 주소) |
| `qor`  | 읽거나 쓸 레지스터의 개수 (Quantity of Registers) |

---

## 🔹 2. 예시: `sendMsgForEmitClean`

```python
sendMsgForEmitClean(
    id=1,
    func=16,
    address=4200,
    qor=8,
    data_length=16,
    data_command=command,
    data_scent_no=scent_no,
    data_scent_pump_power=scent_pump_power,
    ...
)
```

- 장치 1번에게, 4200번 주소부터 8개 레지스터에 대해 다중 쓰기(16) 명령을 전송

---

## 🔹 3. 함수별 목적 및 차이

| 함수명 | 기능 요약 |
|--------|-----------|
| `sendMsgWriteSingleRegister` | 단일 레지스터 쓰기 (MODBUS function code 6) |
| `sendMsgForEmitClean` | 다중 레지스터 쓰기 (MODBUS function code 16) |

### 📌 상세 비교

| 항목     | `sendMsgWriteSingleRegister` | `sendMsgForEmitClean` |
|----------|------------------------------|------------------------|
| 명령 타입 | 단일 쓰기                     | 다중 쓰기               |
| Function Code | 6 (Write Single Register) | 16 (Write Multiple Registers) |
| 메시지 길이 | 짧음 (고정 7바이트 수준)     | 김 (다수 파라미터 포함)     |
| 사용 목적 | 정지, 간단 설정 변경             | 발향, 세정 등 복합 제어       |
| 호출 시점 | 단순 명령 전달 시               | 복합 명령 전송 시           |

---

## 🔹 4. 패킷 구조 비교

### ✅ `sendMsgWriteSingleRegister`

```
[ID] [FUNC] [ADDRESS(2B)] [DATA(2B)] [CRC(2B)]
```

- 간단한 명령. 예: 01 06 10 74 01 90 CRC16

### ✅ `sendMsgForEmitClean`

```
[ID] [FUNC] [ADDRESS(2B)] [QOR(2B)] [DATA_LENGTH(1B)] 
[COMMAND(2B)] [SCENT_NO(2B)] [SCENT_PUMP_POWER(2B)] 
[CLEAN_PUMP_POWER(2B)] [SCENT_PERIOD(2B)] 
[CLEAN_PERIOD(2B)] [SCENT_DELAY(2B)] 
[CLEANUP_DELAY(2B)] [CRC(2B)]
```

- 복잡한 명령 구성. 다수 파라미터 포함

---

## ✅ 결론 요약

| 항목                 | 단일 쓰기                             | 다중 쓰기                         |
|--------------------|--------------------------------------|----------------------------------|
| 함수명               | `sendMsgWriteSingleRegister`         | `sendMsgForEmitClean`           |
| function code       | 6                                    | 16                               |
| 메시지 구조          | 단순, 7바이트 내외                    | 복잡, 20바이트 이상               |
| 쓰기 대상            | 레지스터 1개                          | 레지스터 여러 개                  |
| 주요 용도            | 정지, 단일 설정                        | 발향, 세정, 시간/파워/딜레이 등 복합 설정 |
