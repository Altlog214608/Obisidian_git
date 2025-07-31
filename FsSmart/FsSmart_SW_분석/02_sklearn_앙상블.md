

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

다음은 **Bagging, Boosting, Voting, Stacking, XGBoost** 등 주요 앙상블 기법을 구조적으로 정리하고, 이미지로 시각화한 Markdown 문서입니다. Obsidian에서도 바로 활용할 수 있습니다.

[![利用Stacking与XGBoost提升分类性能的实战案例-CSDN博客](https://tse1.mm.bing.net/th/id/OIP.4ybgcSuZqQ0L7qkRZiIR9QHaEL?pid=Api)](https://blog.csdn.net/weixin_42524864/article/details/143444257)

---

## 🧩 앙상블 기법 간 비교 요약

|기법|학습 방식|특징|출력 결합 방식|
|---|---|---|---|
|**Bagging**|병렬+Bootstrap|모델 간 상관성 낮춤, 분산 감소|투표(Voting) / 평균(Averaging)|
|**Random Forest**|Bagging + 랜덤 feature 선택|휘황한 트리 다양성, 과적합 완화|투표 / 평균|
|**Boosting (AdaBoost/GBDT)**|순차적 학습, 오답 중심|오류에 집중해 정확도 향상|weighted sum|
|**Voting Ensemble**|서로 다른 모델 독립 학습|간단한 모델 조합, 안정성 향상|Hard/Soft Voting|
|**Stacking**|다양한 베이스 모델 + 메타 모델|베이스 모델의 예측을 학습해 최종 결정|메타모델 기반 학습|
|**XGBoost**|GBDT 기반 최적화 구현|정규화, 병렬처리, 빠르고 정확함|boosting 합산|

---

## 🔍 각 기법 설명

### 1. Bagging (Bootstrap Aggregating)

- 동일 학습기를 각기 다른 bootstrap 샘플로 병렬 학습
    
- 결과 투표 또는 평균 집계 방식으로 예측
    
- 과적합 감소 및 안정성 확보 ([위키백과](https://en.wikipedia.org/wiki/Ensemble_learning?utm_source=chatgpt.com "Ensemble learning"), [Medium](https://medium.com/%40minhducnguyen20/ensembling-voting-and-stacking-fd0d37ef9791?utm_source=chatgpt.com "Ensembling: Voting and Stacking - Medium"), [fall-2023-python-programming-for-data-science.readthedocs.io](https://fall-2023-python-programming-for-data-science.readthedocs.io/en/latest/Lectures/Theme_3-Model_Engineering/Lecture_14-Ensemble_Methods/Lecture_14-Ensemble_Methods.html?utm_source=chatgpt.com "Lecture 14 - Ensemble Methods"))
    

### 2. Random Forest

- Bagging 기반이나 노드마다 랜덤 feature 서브셋 사용
    
- 트리 간 상관성 감소, 과적합 더 효과적 완화 ([위키백과](https://en.wikipedia.org/wiki/Ensemble_learning?utm_source=chatgpt.com "Ensemble learning"), [Analytics Vidhya](https://www.analyticsvidhya.com/blog/2023/01/ensemble-learning-methods-bagging-boosting-and-stacking/?utm_source=chatgpt.com "Bagging, Boosting and Stacking: Ensemble Learning in ML Models"))
    

### 3. Boosting (AdaBoost, Gradient Boosting, XGBoost 등)

- 이전 모델의 오류에 가중치를 두고 순차적으로 학습
    
- 편향(bias) 감소, 높은 예측력 지님
    
- 다만 과적합 위험 존재 ([interviewnode.com](https://www.interviewnode.com/post/ensemble-learning-techniques-boosting-bagging-and-stacking-explained?utm_source=chatgpt.com "Ensemble Learning Techniques: Boosting, Bagging, and Stacking ..."), [Analytics Vidhya](https://www.analyticsvidhya.com/blog/2023/01/ensemble-learning-methods-bagging-boosting-and-stacking/?utm_source=chatgpt.com "Bagging, Boosting and Stacking: Ensemble Learning in ML Models"))
    

### 4. Voting (Hard / Soft)

- 서로 다른 분류기들의 예측 결과를 투표 또는 평균 방식으로 조합
    
- 단순하지만 효과적인 성능 안정화 방식 ([GeeksforGeeks](https://www.geeksforgeeks.org/ensemble-methods-in-python/?utm_source=chatgpt.com "Ensemble Methods in Python | GeeksforGeeks"), [arXiv](https://arxiv.org/abs/2005.01575?utm_source=chatgpt.com "StackGenVis: Alignment of Data, Algorithms, and Models for Stacking Ensemble Learning Using Performance Metrics"))
    

### 5. Stacking (Stacked Generalization)

- 서로 다른 base 모델들의 예측 결과를 meta learner가 학습
    
- 최종 메타 모델이 각 base 모델의 출력 조합 → 더 높은 성능 기대 ([kaggle.com](https://www.kaggle.com/code/yug201/stacking-ensemble-xgboost-base-catboost-meta?utm_source=chatgpt.com "Stacking Ensemble: XGBoost Base & CatBoost Meta - Kaggle"))
    

### 6. XGBoost (Extreme Gradient Boosting)

- Gradient Boosting 기반 최적화 구현체
    
- 정규화, 병렬 학습, 자동 missing 처리, 빠른 실행 등이 특징
    
- 실전 데이터셋에서 뛰어난 성능 보임 ([xgboosting.com](https://xgboosting.com/stacking-ensemble-with-xgboost-meta-model-final-model/?utm_source=chatgpt.com "Stacking Ensemble With XGBoost Meta Model (Final Model)"), [Medium](https://medium.com/%40bhatadithya54764118/day-41-ensemble-learning-practical-stacking-and-voting-classifiers-366b5fb64616?utm_source=chatgpt.com "Ensemble Learning Practical — Stacking and Voting Classifiers | by ..."))
    

---

## 🧠 앙상블 기법 간 상관 관계

- **Random Forest**는 Bagging의 확장
    
- **XGBoost**는 Boosting 기반 고도화된 구조
    
- **Voting**은 개별 모델을 그대로 결합하는 방식
    
- **Stacking**은 모델 간 협업된 예측을 학습하는 고도 앙상블
    

---

## 📘 참고 이미지 설명

- 🔺 왼쪽 상단: stacking 구조—베이스 모델과 메타 모델의 예측 결합 흐름
    
- 🔺 오른쪽 상단: 다양한 분류기 예측 결과의 voting 구조 ([Medium](https://medium.com/%40minhducnguyen20/ensembling-voting-and-stacking-fd0d37ef9791?utm_source=chatgpt.com "Ensembling: Voting and Stacking - Medium"), [Medium](https://medium.com/sfu-cspmp/xgboost-a-deep-dive-into-boosting-f06c9c41349?utm_source=chatgpt.com "XGBoost: A Deep Dive into Boosting | by Rohan Harode | SFU Professional ..."))
    
- 🔺 하단: stacking 개요 다이어그램—기존 예측 결과를 메타 모델이 재학습하는 방식
    

---

필요하시면 **AdaBoost, XGBoost 내부 구조**, 또는 **VotingClassifier / StackingClassifier 사용 예제 코드**도 제공해 드릴 수 있어요. 필요하시면 말씀해 주세요!