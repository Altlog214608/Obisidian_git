
---

## 🏷️ 앙상블 기법별 구조 및 설명

[![Ensemble Learning Methods: Bagging, Boosting and Stacking](https://tse4.mm.bing.net/th/id/OIP.a6hnuJ8WM37mLimHfMORmQHaDq?pid=Api)](https://www.analyticsvidhya.com/blog/2023/01/ensemble-learning-methods-bagging-boosting-and-stacking/)

### **1. Bagging (Bootstrap Aggregating)**

- **구조**: 동일 모델(예: 약한 학습기)을 다양한 랜덤 샘플로 병렬 학습
    
- **결합 방식**: 분류 → 다수결 투표 / 회귀 → 평균
    
- **효과**: 분산(variance) 감소로 예측 안정성 향상, 과적합 억제
    
- **대표 모델**: Random Forest
    
- **단점**: 편향(bias)은 줄이지 않음  
    ([GeeksforGeeks](https://www.geeksforgeeks.org/machine-learning/a-comprehensive-guide-to-ensemble-learning/?utm_source=chatgpt.com "Ensemble Learning - GeeksforGeeks"), [위키백과](https://en.wikipedia.org/wiki/Bootstrap_aggregating?utm_source=chatgpt.com "Bootstrap aggregating"))
    

---

[![Ensemble Learning Methods: Bagging, Boosting and Stacking](https://tse3.mm.bing.net/th/id/OIP.4XuD6oRrgVqtaSwH-cu6SAHaDq?pid=Api)](https://www.analyticsvidhya.com/blog/2023/01/ensemble-learning-methods-bagging-boosting-and-stacking/)

### **2. Boosting (AdaBoost, GBDT 등)**

- **구조**: 순차적 학습. 이전 약한 모델이 틀린 샘플에 높은 가중치 부여
    
- **결합 방식**: 각 모델 예측의 가중 합 (weighted sum)
    
- **효과**: 편향(bias) 최소화, 예측 정확도 상승
    
- **단점**: 과적합 가능성, 순차 학습 → 느림  
    ([위키백과](https://en.wikipedia.org/wiki/Boosting_%28machine_learning%29?utm_source=chatgpt.com "Boosting (machine learning)"), [Cross Validated](https://stats.stackexchange.com/questions/552356/boosting-reduces-bias-when-compared-to-what-algorithm?utm_source=chatgpt.com "Boosting reduces bias when compared to what algorithm?"), [Analytics Vidhya](https://www.analyticsvidhya.com/blog/2023/01/ensemble-learning-methods-bagging-boosting-and-stacking/?utm_source=chatgpt.com "Bagging, Boosting and Stacking: Ensemble Learning in ML Models"))
    

---
![[Pasted image 20250731171512.jpg]]

### **3. Stacking (Stacked Generalization)**

- **구조**: 서로 다른 베이스 모델(B, C, D)의 예측 결과를 **메타 모델**이 학습
    
- **결합 방식**: meta learner가 base 모델 예측들을 feature로 받아 최종 예측
    
- **효과**: 서로 다른 모델의 장점을 결합해 일반화 성능 향상
    
- **단점**: 구성 복잡, 과적합 위험 있음  
    ([Medium](https://medium.com/%40abhishekjainindore24/different-types-of-ensemble-techniques-bagging-boosting-stacking-voting-blending-b04355a03c93?utm_source=chatgpt.com "Bagging, Boosting, Stacking, Voting, Blending | by Abhishek Jain"))
    

---

[![The structure of the extreme gradient boosting (XGBoost) approach ...](https://tse2.mm.bing.net/th/id/OIP.gzAo1kBkzVf9ffoHH_kOwQHaEj?pid=Api)](https://www.researchgate.net/figure/The-structure-of-the-extreme-gradient-boosting-XGBoost-approach_fig7_370857103)

### **4. XGBoost (Extreme Gradient Boosting)**

- **구조**: Gradient Boosting의 고도화 구현체. 트리 기반 순차 학습
    
- **특징**: 정규화, 병렬 처리, 결측치 자동 처리, 과적합 방지 기능 내장
    
- **효과**: GBDT보다 빠르고 정확하며 실무에서 널리 사용
    
- **단점**: 하이퍼파라미터 튜닝 복잡  
    ([Medium](https://medium.com/%40patwariraghottam/ensemble-learning-unveiled-comparing-bagging-boosting-voting-and-stacking-with-xgboost-leading-eab2de794c16?utm_source=chatgpt.com "Ensemble Learning Unveiled: Comparing Bagging, Boosting, Voting ..."), [Analytics Vidhya](https://www.analyticsvidhya.com/blog/2023/01/ensemble-learning-methods-bagging-boosting-and-stacking/?utm_source=chatgpt.com "Bagging, Boosting and Stacking: Ensemble Learning in ML Models"), [위키백과](https://en.wikipedia.org/wiki/Boosting_%28machine_learning%29?utm_source=chatgpt.com "Boosting (machine learning)"))
    

---

## 🧾 모델 비교 정리

|기법|학습 방식|결합 방식|주요 효과|단점|
|---|---|---|---|---|
|**Bagging**|병렬 학습 (bootstrap)|투표 / 평균|분산 감소, 안정성 향상|편향은 줄이지 못함|
|**Boosting**|순차 학습 (오류 중심)|가중 합|편향 감소, 정확도 상승|과적합 가능성, 학습 속도 느림|
|**Stacking**|병렬 + meta 학습|메타 모델 기반 결합|각 모델 장점 결합, 일반화 우수|구현 복잡, 과적합 위험 있음|
|**XGBoost**|Gradient Boosting 최적화|Boosting 합산|빠르고 정확한 부스팅 구현체|설정 복잡, 초기 학습 비용 높음|

---

## 🔍 요약

- **Bagging**: 동일 모델을 여러 샘플로 병렬 학습 → 과적합 억제, 안정성 확보
    
- **Random Forest**: Bagging의 확장 (랜덤 feature 추출 포함)
    
- **Boosting**: 순차적 학습으로 약한 학습기의 단점 보완 → 높은 예측 성능
    
- **Stacking**: 다양한 모델을 결합해 메타 학습기로 최적화된 예측 생성
    
- **XGBoost**: 실제 응용에서 성능/속도 최적화된 Boosting 구현체
    

