

아래는 주요 앙상블 기법(Bagging, Boosting, Stacking, XGBoost)을 **모델별로 각각 한 장의 시각 이미지**와 설명을 포함해 Markdown 형식으로 정리한 내용입니다. Obsidian에 그대로 붙여넣으면 바로 활용 가능합니다.

---

## 🖼️ Bagging

[![Demystifying Ensemble Methods: Boosting, Bagging, and Stacking ...](https://tse2.mm.bing.net/th/id/OIP.9sHDQ5Jf2DhcjaXYgxcwsgHaES?r=0&pid=Api)](https://machinelearningmastery.com/demystifying-ensemble-methods-boosting-bagging-and-stacking-explained/)

**Bagging (Bootstrap Aggregating)**

- 동일한 학습기를 여러 부트스트랩 샘플로 병렬 학습
    
- 각 모델의 예측을 **투표 또는 평균으로 결합**
    
- 과적합 감소 · 예측 안정성 향상에 효과적 ([MachineLearningMastery.com](https://machinelearningmastery.com/demystifying-ensemble-methods-boosting-bagging-and-stacking-explained/?utm_source=chatgpt.com "Demystifying Ensemble Methods: Boosting, Bagging, and Stacking ..."), [Jun’s Archive](https://esj205.oopy.io/dd0a89f9-1cac-4578-a3a1-7bb458836311?utm_source=chatgpt.com "Bagging vs Boosting vs Stacking"))
    

---

## 🖼️ Boosting

[![Ensemble Methods: dé 3 methoden eenvoudig uitgelegd](https://tse3.mm.bing.net/th/id/OIP.6mX3aeRad1-yC7_VWQvhqQHaEw?r=0&pid=Api)](https://pythoncursus.nl/ensemble-methods/)

**Boosting (AdaBoost, GBDT 등)**

- 약한 학습기를 **순차적으로 학습**
    
- 이전 단계의 오류 사례에 가중치를 주어 집중 학습
    
- 예측은 **학습기별 결과에 가중합** 방식으로 최종 결정 ([Jun’s Archive](https://esj205.oopy.io/dd0a89f9-1cac-4578-a3a1-7bb458836311?utm_source=chatgpt.com "Bagging vs Boosting vs Stacking"))
    

---

## 🖼️ Random Forest / Stacking

[![Bagging vs Boosting vs Stacking](https://tse2.mm.bing.net/th/id/OIP.T0QxT02mrNg6jqcU2XJb6wHaEL?r=0&pid=Api)](https://esj205.oopy.io/dd0a89f9-1cac-4578-a3a1-7bb458836311)

- **Random Forest**는 Bagging 기반이지만, 각 트리가 다른 특성(feature subset)만 사용해 학습 → 모델 간 상관성 감소 → 투표 방식 결합
    
- **Stacking**은 서로 다른 모델(B, C, D 등)의 예측 결과들을 모아서 **Meta-learner**가 추가 학습 → 최종 예측 생성 ([Medium](https://medium.com/%40patwariraghottam/ensemble-learning-unveiled-comparing-bagging-boosting-voting-and-stacking-with-xgboost-leading-eab2de794c16?utm_source=chatgpt.com "Ensemble Learning Unveiled: Comparing Bagging, Boosting, Voting ..."), [Jun’s Archive](https://esj205.oopy.io/dd0a89f9-1cac-4578-a3a1-7bb458836311?utm_source=chatgpt.com "Bagging vs Boosting vs Stacking"))
    

---

## 🖼️ XGBoost

[![XGBoost Explained: A Beginner’s Guide | by Jamie Crossman-Smith | Low ...](https://tse1.mm.bing.net/th/id/OIP.bGrSU6jjt_KSo_DfdSIVcwHaDe?r=0&pid=Api)](https://medium.com/low-code-for-advanced-data-science/xgboost-explained-a-beginners-guide-095464ad418f)

**XGBoost (Extreme Gradient Boosting)**

- Gradient Boosting 기반의 고성능 모델
    
- **정규화, 병렬 처리, 결측치 자동 처리**, 빠른 학습 속도 등의 고급 기능 포함
    
- Boosting의 순차 방식에 최적화된 구현체로, 실무에서 뛰어난 성능 발휘 ([Scikit-learn](https://scikit-learn.org/stable/modules/ensemble.html?utm_source=chatgpt.com "1.11. Ensembles: Gradient boosting, random forests, bagging, voting ..."), [MachineLearningMastery.com](https://machinelearningmastery.com/demystifying-ensemble-methods-boosting-bagging-and-stacking-explained/?utm_source=chatgpt.com "Demystifying Ensemble Methods: Boosting, Bagging, and Stacking ..."))
    

---

## ⚖️ 앙상블 기법 비교 요약 표

|모델|설명 요약|
|---|---|
|Bagging|병렬 학습 + 투표/평균 결합 → 분산 감소|
|Random Forest|Bagging + 랜덤 feature 선택 → 안정성 및 정확도 증가|
|Boosting|순차 학습 + 가중합 결합 → 오류 중심 학습, 정확도 향상|
|Stacking|다양한 베이스 모델 예측 결과를 meta 모델이 추가 학습|
|XGBoost|고도화된 Boosting 구현체, 빠르고 정교한 예측 가능|

---

필요하시면 **VotingClassifier, AdaBoost 사용 예제**, 혹은 **각 기법별 장단점 목록**, **Scikit‑learn 코드 샘플**도 제공해 드릴 수 있어요!