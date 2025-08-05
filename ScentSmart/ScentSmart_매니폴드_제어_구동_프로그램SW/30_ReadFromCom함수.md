아래는 `ReadFromCom()` 함수의 내부 동작을 **초보자도 이해할 수 있도록 한 줄씩 상세하게 풀이**한 설명입니다.

## ReadFromCom 함수 구조 (핵심 부분 인용)

c

```c
int ReadFromCom(struct s_SERIAL *scom) {     char checkcrc, message[64];    int i, result, inqlen;    Delay(0.05);     if (scom->terminator) {        scom->readbyte = ComRdTerm(scom->port, scom->read, SIZEBUFFER-2, scom->terminator);    }    else {        inqlen = GetInQLen(scom->port);        if (inqlen) {            ComRd(scom->port, scom->read, inqlen);            scom->read[inqlen] = '\0';            scom->readbyte = inqlen;        }    }     // (중간에 debug용 출력 생략)    scom->error = ReturnRS232Err();     switch (scom->crc) {        case 1 : checkcrc = CheckCRC16(scom->sendbyte,scom->send); break;        case 2 : checkcrc = CheckModbusCRC(scom->sendbyte,scom->send); break;        default: break;    }     if (scom->crc && checkcrc) {        sprintf(message,"CRC error!");        MessagePopup("Error",message);        return (checkcrc);    }     if (!scom->error) DecodePacket();    return (scom->error); }
```

## 1. **함수 선언부/지역 변수**

- `int ReadFromCom(struct s_SERIAL *scom)`
    
    - 이 함수는 "시리얼 포트에서 데이터를 읽는" 역할입니다. `s_SERIAL` 구조체 포인터(장비 상태/통신정보를 담는 구조체)를 인자로 받음.
        
- 내부 변수들:
    
    - `char checkcrc, message;` → CRC 검사와 에러 메시지 용
        
    - `int i, result, inqlen;` → 반복, 결과, 입력 버퍼 길이 저장
        

## 2. **Delay(0.05);**

- 아주 짧게(약 50ms) 대기합니다.
    
- 이유: 통신 안정성, 데이터가 들어오는 시간(하드웨어 입장에선, PC가 너무 빨리 읽으려 하면 데이터가 완전히 안 들어온 상태일 수 있기 때문)
    

## 3. **데이터 읽기**

## 1) Terminator 방식인지 확인

- `if (scom->terminator) { ... }`
    
    - 패킷의 끝(terminator, 예를 들면 `\n` 또는 지정한 문자)로 패킷 경계를 구분한 경우 사용
        
    - `"ComRdTerm"` 함수: 지정한 종료문자(terminator)가 나올 때까지 읽어서 `scom->read` 버퍼에 저장
        
    - 읽은 총 바이트 수는 `scom->readbyte`에 입력
        

## 2) 아니면, 남은 데이터 길이만큼 읽기

- `else { ... }`
    
    - terminator를 안 쓰는 경우, 남은 데이터 길이만큼 한 번에 쫘악 읽음
        
    - `inqlen = GetInQLen(scom->port)`로 "내부 받아온 데이터의 남은 길이" 측정
        
    - `ComRd` 함수: inqlen 만큼 데이터 받아와 `scom->read`에 저장
        
    - `scom->read[inqlen] = '\0';` : 텍스트 처리(문자열 후단에 종료문자 0 넣기, 가끔 버그 방지용)
        
    - 그 길이를 `scom->readbyte`에 저장
        

## 4. **(생략) Debug 출력**

- 옵션에 따라 읽어온 바이트들을 HEX(16진수)나 문자열로 printf() 해서 로그로 찍을 수 있음.
    
- 실전에서는 없어도 됨, 입문자라면 그냥 어떻게 데이터가 보이는지 확인 용도
    

## 5. **시리얼 에러 상태 체크**

- `scom->error = ReturnRS232Err();`
    
    - 통신과정에서 에러 발생(예: 포트 닫힘, 파라미터 틀림, 등) 여부를 확인해서 에러코드로 저장
        

## 6. **CRC 체크(패킷 무결성 검사)**

- switch (scom->crc)
    
    - `scom->crc == 1`이면 `CheckCRC16` 함수 사용, `2`이면 `CheckModbusCRC` 사용(장비 통신 규약에 따라 다름)
        
    - CRC 함수는 "패킷이 전송 도중 깨지지 않았는지" 검사(틀리면 -1, 맞으면 0)
        
    - 만약 CRC 체크에 실패하면
        
        - `MessagePopup("Error",message);`
            
        - "CRC error!" 라는 에러창 띄우고 리턴
            

## 7. **정상 데이터 해석 요청**

- `if (!scom->error) DecodePacket();`
    
    - 에러가 없으면
        
    - 모든 바이트 정보가 들어있는 `scom->read` 배열을 해석하는 `DecodePacket()` 함수 호출(실제 패킷 구조 해석 및 동작별 분기 여기서 진행)
        

## 8. **return (scom->error);**

- 함수의 리턴값은 "0(정상)" 또는 에러코드(음수 등)
    
- 이걸로 상위로 결과를 알려줌
    

# **정리**

|단계|하는 일(쉽게 설명)|
|---|---|
|변수 선언|검사/저장용 변수 준비(수신 데이터, 에러, CRC 등)|
|지연(Delay)|데이터가 다 들어올 때까지 0.05초 잠깐 기다림|
|데이터 읽기|terminator 방식이면 해당 문자가 나올 때까지, 아니면 남은 데이터 전부를 read 버퍼에 저장|
|에러 체크|시리얼 포트의 에러가 있는지 확인함|
|CRC 체크|데이터 깨진 거 없는지 검사. 문제 있으면 에러 메시지 띄움|
|패킷해석|정상적이면 실제 '어떤 명령이 왔는지' 분리 해석 (`DecodePacket()` 함수에서 처리)|
|리턴|실행 결과(에러코드 또는 0) 상위로 알려줌|

## 💡 요약 콘셉트

- **패킷을 받아서, 깨짐/오류유무 검사(CRC, 에러코드 등)**
    
- **실제로는 scom->read 배열에 Python이 보낸 바이트 순서대로 저장**
    
- **정상 패킷이면 DecodePacket에서 명령/데이터 분리해서 실제 동작(모터, 펌프 등)에 연결**
    
- **이 과정 중 어디서든 오류나면 바로 중단/에러리턴**
    

궁금하면 더 자세하게 각 함수, 각 버퍼, C↔Python 바이트 동작을 추가로 설명드릴 수 있습니다!

1. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/6ea26fc2-9e2b-43eb-9434-7d868e463f83/ConfigCOM.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/6ea26fc2-9e2b-43eb-9434-7d868e463f83/ConfigCOM.c)
2. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/e4f528e1-73e2-4306-9af6-7bd10a292b6c/ConfigCOM.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/e4f528e1-73e2-4306-9af6-7bd10a292b6c/ConfigCOM.h)
3. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/311793f0-00ec-4937-9ab1-0ca0be611e23/ControlPumpNF.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/311793f0-00ec-4937-9ab1-0ca0be611e23/ControlPumpNF.c)
4. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/50ffaa46-660c-4eee-ac44-5a3b310f5c38/ControlPumpNF.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/50ffaa46-660c-4eee-ac44-5a3b310f5c38/ControlPumpNF.h)
5. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/f4b9368d-2bbe-4c48-8f5f-77356ed069f0/FunctionSerial.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/f4b9368d-2bbe-4c48-8f5f-77356ed069f0/FunctionSerial.c)
6. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/4c6a2472-7ee6-48a7-baa6-d98a97a1657d/FunctionSerial.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/4c6a2472-7ee6-48a7-baa6-d98a97a1657d/FunctionSerial.h)
7. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/0734a919-c2fc-4105-b173-48625c60b5f3/MODBUS.c](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/0734a919-c2fc-4105-b173-48625c60b5f3/MODBUS.c)
8. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/ad82efcb-cc4e-43b2-839c-0603955dc00e/MODBUS.h](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/ad82efcb-cc4e-43b2-839c-0603955dc00e/MODBUS.h)
9. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/11dbb366-8707-4401-ae96-0b74a041019d/dsComm.py](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/11dbb366-8707-4401-ae96-0b74a041019d/dsComm.py)
10. [https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d9dac957-04db-41bb-af5e-2f6c9baf3bbc/FrSmart.py](https://ppl-ai-file-upload.s3.amazonaws.com/web/direct-files/attachments/72041696/d9dac957-04db-41bb-af5e-2f6c9baf3bbc/FrSmart.py)