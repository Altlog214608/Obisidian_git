
---

````markdown
# 🤖 FrSmart 향수 추천 AI 모델 분석 (testModel.py 기반)

FrSmart에서는 **사용자 조건(나이, 성별, 성격, 취향 등)**을 입력받아 향수를 추천하는 **랜덤포레스트 기반 분류 모델**을 사용합니다.

---

## 🔍 1. 모델 개요

- **모델 종류:**  
  - `RandomForestClassifier` (scikit-learn)
- **추천 대상:**  
  - 향수 ID (Recommend_Perfume_Id)
- **입력 특성 (Features):**
  | 특성 | 설명 |
  |------|------|
  | Age | 연령 (10단위 정수화) |
  | Gender | 성별 ('Man' / 'Woman') |
  | Personality | 성격 (예: 'Extraversion') |
  | Preferred_Scent | 선호 향기 (예: 'Floral', 'Woody') |
  | Preferred_Color | 선호 색상 (예: 'Red', 'Blue') |
  | Price | 가격대 ('Entry', 'Middle', 'Highend') |

---

## ⚙️ 2. 데이터 구성 및 전처리

- **데이터 파일:** `perfume_training_df.csv`
- **전처리 방식:**
  - `OneHotEncoder`: 범주형 특성 처리
  - `ColumnTransformer`: 여러 컬럼에 OHE 적용
- **변환된 결과:**  
  범주형 데이터를 모델 학습에 사용 가능한 벡터로 변환

---

## 🧱 3. 학습 파이프라인 구조

```python
pipeline = Pipeline([
    ('preprocessor', ColumnTransformer([... OneHotEncoder ...])),
    ('classifier', RandomForestClassifier(...))
])
````

- 파이프라인 구성:
    
    1. 범주형 인코딩 처리
        
    2. 랜덤포레스트 분류기 적용
        
- `train_test_split()`으로 학습/테스트 데이터 분리 후 `.fit()` 학습 수행
    

---

## 📥 4. 추천 예측 흐름

- **입력:**  
    UI 선택 결과 → dict 형태로 구성된 추천 입력값
    

```python
find_input_data = {
   'Age': age * 10,
   'Gender': dsNum.find_gender_text[gender],
   'Personality': dsNum.find_personality_text[personality],
   ...
}
```

- **예측 호출:**
    

```python
result = testModel.find_scent(find_input_data)
predicted_id = result['predicted_id'] + 1  # 1-indexed
```

- **출력:**  
    추천 향수 ID + 예측 신뢰도(confidence)
    

---

## 🔗 5. 모델 동작 방식 요약

|단계|설명|
|---|---|
|1️⃣ 사용자 입력|나이/성별/성격/취향 등 선택|
|2️⃣ 입력 데이터 전처리|OneHot 인코딩|
|3️⃣ 모델 학습 & 예측|RandomForestClassifier.fit() & predict()|
|4️⃣ 추천 결과 표시|추천 향수 ID + 신뢰도 UI 반영 및 QR 출력|

---

## 🧾 요약

- **분류 문제:** 향수 ID 추천
    
- **모델 구조:** OneHotEncoded categorical input → RandomForestClassifier
    
- **실시간 처리:** 예측시마다 모델 재학습 (속도는 데이터 크기에 따라 달라짐)
    
- **UI 연결:** UI → dict 변환 → 모델 호출 → 결과 출력 & 향기 발향 연결
    

---

## 📁 관련 파일

- `testModel.py`: 추천 알고리즘 전체 로직 포함
    
- `perfume_training_df.csv`: 학습 데이터 파일
    
- `FrSmart.py`: UI 이벤트에서 추천 모델 호출 및 결과 연결
    

---
