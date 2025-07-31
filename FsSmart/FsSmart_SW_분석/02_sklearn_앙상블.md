

[![Ensemble: Bagging, Random Forest, Boosting and Stacking](https://tse4.mm.bing.net/th/id/OIP.Q3_x_sty9Vct1kiineS8yAHaGw?pid=Api)](https://tungmphung.com/ensemble-bagging-random-forest-boosting-and-stacking/)

---

## 🧩 앙상블 방법 간 비교

### 1. Bagging (Bootstrap Aggregating)

- **독립 모델 병렬 학습**
    
    - training 데이터를 부트스트랩(복원추출)으로 여러 번 샘플링
        
    - 각 샘플마다 모델(예: 결정트리)을 별도로 학습
        
- **결과 결합 방식**
    
    - 분류: 다수결 투표(voting)
        
    - 회귀: 평균(averaging)
        
- **특징**: 모델 간 상관성 감소, 분산 감소 → 과적합 완화  
    ([위키백과](https://en.wikipedia.org/wiki/Bootstrap_aggregating?utm_source=chatgpt.com "Bootstrap aggregating"), [위키백과](https://en.wikipedia.org/wiki/Out-of-bag_error?utm_source=chatgpt.com "Out-of-bag error"), [Cross Validated](https://stats.stackexchange.com/questions/365437/what-are-advantages-of-random-forests-vs-using-bagging-with-other-classifiers?utm_source=chatgpt.com "What are advantages of random forests vs using bagging with other ..."), [위키백과](https://en.wikipedia.org/wiki/Random_forest?utm_source=chatgpt.com "Random forest"), [위키백과](https://en.wikipedia.org/wiki/Ensemble_learning?utm_source=chatgpt.com "Ensemble learning"))
    

---

### 2. Random Forest

- Bagging 기반이지만, 각 트리 학습 시 **특징 랜덤 선택(feature subset)** 추가
    
    - 노드마다 무작위 feature 일부만 고려하여 분할
        
- 결과적으로 트리 간 상관성 감소 → Bagging보다 더 효과적
    
- **결과 결합**: Bagging과 동일하게 voting / averaging  
    ([Cross Validated](https://stats.stackexchange.com/questions/365437/what-are-advantages-of-random-forests-vs-using-bagging-with-other-classifiers?utm_source=chatgpt.com "What are advantages of random forests vs using bagging with other ..."), [위키백과](https://en.wikipedia.org/wiki/Random_forest?utm_source=chatgpt.com "Random forest"))
    

---

### 3. Boosting

- **순차적 모델 학습**
    
    - 이전 모델이 잘못 분류한 사례에 가중치를 높여 다음 모델 학습
        
    - 대표 알고리즘: AdaBoost, Gradient Boosting (GBDT), XGBoost 등
        
- **결과 결합 방식**
    
    - 각 약한 학습기(weak learner)의 예측을 **가중합(weighted sum)**
        
- **특징**: 편향을 줄여 정확도 향상 → 과적합 가능성 있음  
    ([위키백과](https://en.wikipedia.org/wiki/Boosting_%28machine_learning%29?utm_source=chatgpt.com "Boosting (machine learning)"), [thechangelab.stanford.edu](https://thechangelab.stanford.edu/tutorials/longitudinal-design-data-analysis/ensemble-methods-bagging-random-forests-boosting/?utm_source=chatgpt.com "Ensemble Methods – Bagging, Random Forests, Boosting"))
    

---

## ⚖️ 비교 요약표

|앙상블 방식|학습 방식|특징|결과 결합 방식|
|---|---|---|---|
|**Bagging**|병렬, 서로 다른 bootstrap 샘플 사용|분산 감소, 안정성 향상|다수결 / 평균|
|**Random Forest**|Bagging + 랜덤 feature 선택|트리 상관성 감소, 더 안정적|투표 / 평균|
|**Boosting**|순차적, 오류 위주로 학습|편향 감소, 정확도 높음|학습기별 가중합|

---

## 📘 참고용 이미지 설명

- **왼쪽 위**: 하나의 데이터를 여러 샘플로 나누고 트리를 병렬로 학습한 후 평균 또는 투표 집계 (Bagging 개념)
    
- **오른쪽 위**: Boosting 흐름도 — 이전 오류 중심으로 다음 모델이 학습되는 순차 구조
    
- **왼쪽 아래**: 앙상블 전체 구조 개요 — 다양한 모델 조합 흐름도
    
- **오른쪽 아래**: Boosting 방법 강조 구조 — 데이터별 학습 순서 표현
    

---

## 🧠 왜 앙상블이 효과적인가?

- **Bagging**: 과적합(overfitting)을 줄이면서 분산(variance)을 감소
    
- **Boosting**: 낮은 성능의 weak learner를 점진적으로 개선하여 bias 감소
    
- **Random Forest**: Bagging + Random Feature 선택으로 모델 다양성을 증가시키며 안정성과 정확도 모두 향상  
    ([analyticsvidhya.com](https://www.analyticsvidhya.com/blog/2023/01/ensemble-learning-methods-bagging-boosting-and-stacking/?utm_source=chatgpt.com "Bagging, Boosting and Stacking: Ensemble Learning in ML Models"), [geeksforgeeks.org](https://www.geeksforgeeks.org/machine-learning/bagging-vs-boosting-in-machine-learning/?utm_source=chatgpt.com "Bagging vs Boosting in Machine Learning - GeeksforGeeks"))
    

---

원하시면 **Stacking**, **Voting**, **AdaBoost**, **XGBoost** 등 다른 앙상블 기법 비교도 추가해드릴 수 있습니다. 필요하시면 말씀해주세요!