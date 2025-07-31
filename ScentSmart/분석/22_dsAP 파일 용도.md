

---

````markdown
# 🔐 dsAP.json 파일 구조 및 용도 분석

`dsAP` 파일은 프로그램 실행 시 사용자의 **접근 권한 제어** 및 **비밀번호 로그인 기능**을 위한 설정 파일로 사용됩니다.

---

## 📄 파일 예시

```json
{
    "APC": 0,
    "AP": "gAAAAAAAAAAAvcAsFofXR7XlzNv5B6-g-qylqBbH56wpGlBxGMZUNTITc9Nmm6Bg2euaWNsETZjvygOHleRgv6-D-UKVbScOgg=="
}
````

---

## 🔍 필드 설명

|필드명|의미|설명|
|---|---|---|
|`APC`|Access Password Count 또는 Attempt Password Count|비밀번호 입력 실패 횟수 카운트 (초기값: 0)|
|`AP`|Access Password (암호문)|암호화된 비밀번호 (Fernet 방식, base64 문자열)|

---

## 🔐 사용 목적

- 프로그램 로그인 시 **입력한 비밀번호**를 `dsAP["AP"]`와 비교하여 인증
    
- `dsCrypto.py`의 `decryptMessage()`로 복호화하여 평문 비교
    
- **로그인 실패 횟수(`APC`)를 증가**시켜 보안 강화
    

---

## ⚙️ 작동 흐름

1. **시작 시 로드**
    
    - `dsAP.json` 파일을 로딩
        
    - 암호화된 `"AP"` 값을 `decryptMessage()`로 복호화하여 로그인 비교
        
2. **로그인 시도**
    
    - 입력값과 복호화된 비밀번호 일치 여부 확인
        
    - 실패 시 `"APC"` 값을 증가시켜 차단 로직 가능
        
3. **로그인 성공/변경 시 저장**
    
    - 새 비밀번호 설정 시 `"AP"`에 암호화된 문자열 저장
        
    - `"APC"` 값 초기화 또는 유지
        

---

## 🔗 관련 함수 예시 (`dsCrypto.py` 활용)

```python
# 비밀번호 검증
with open('dsAP.json', 'r') as f:
    ds_ap_data = json.load(f)

stored_encrypted_pw = ds_ap_data['AP']
input_pw = get_user_input()

# 복호화 및 비교
stored_plain_pw = dsCrypto.decryptMessage(stored_encrypted_pw)
if input_pw == stored_plain_pw:
    print("로그인 성공")
else:
    ds_ap_data['APC'] += 1  # 실패 횟수 증가
```

---

## 📌 요약

- `dsAP`는 **로그인 인증 및 보안 관리용 JSON 파일**
    
- `AP`: 암호화된 접근 비밀번호 (Fernet 방식)
    
- `APC`: 비밀번호 실패 횟수 카운터
    
- `dsCrypto.py`와 연동하여 암호화/복호화 처리
    

---

## 📁 위치 및 사용 맥락

- 프로젝트 루트 또는 설정 디렉토리에 저장됨
    
- ScentSmart 프로그램 내 로그인, 비밀번호 변경, 보안 제어에 필수적으로 사용됨
    

```

---

원하시면 이 구조에 맞춰 `dsAP.json`을 생성하거나, 기본 템플릿/초기화 코드를 함께 만들어드릴 수도 있습니다. 요청만 주세요!
```