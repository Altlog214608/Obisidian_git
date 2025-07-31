


---
![Optimized XGBoost Model with Small Dataset for Predicting Relative ...](https://tse2.mm.bing.net/th/id/OIP.YuKnqMuI-ZD0eGwA-or4qAHaEs?pid=Api)
## 🧠 머신러닝 & 딥러닝 모델 정리

### 1. XGBoost Regression

- **개요**: Gradient Boosting 기반의 회귀 모델로, 여러 개의 트리를 순차적으로 학습하며 잔차(residual)를 줄여나갑니다.
    
- **원리**: 처음 트리가 예측 → 잔차 계산 → 다음 트리가 잔차를 보정 → 반복 학습 → 최종 예측.
    
- **활용**: 수치 예측, 피처 중요도 분석 등에 널리 사용됩니다.
    


---
![[Pasted image 20250731175857.jpg]]
### 2. MLP (Multi‑Layer Perceptron)

- **개요**: 입력층 → 은닉층(n개) → 출력층으로 구성된 전형적인 신경망입니다.
    
- **특징**: 모든 노드가 완전 연결(Fully Connected)되어 있으며, 비선형 활성화 함수로 복잡한 패턴을 학습할 수 있습니다.
    
- **활용**: 추천 시스템, 분류, 회귀 등 다양한 일반 머신러닝 문제에 사용됩니다.
    

---
![[Pasted image 20250731175903.jpg]]
### 3. VAE (Variational Autoencoder)

- **개요**: 인코더와 디코더로 구성된 확률 기반 생성 모델로, 잠재 공간(latent space)을 통해 데이터를 압축하고 재구성합니다.
    
- **특징**: 인코더는 입력을 평균(μ), 분산(σ)으로 매핑 → 잠재 벡터 생성(z = μ + σ·ε) → 디코더가 재구성을 수행.
    
- **활용**: 이미지/데이터 생성, 이상치 탐지, 노이즈 제거 등에 활용됩니다.


---

### 4. GAN (Generative Adversarial Network)![[Pasted image 20250731175906.jpg]]

- **개요**: 생성자(Generator)와 판별자(Discriminator)가 적대적으로 경쟁하며 학습하는 생성 모델입니다.
    
- **작동 방식**:
    
    - 생성자: 랜덤 노이즈 → 가짜 샘플 생성
        
    - 판별자: 진짜 vs 가짜 샘플 구분
        
    - 서로 경쟁하며 두 모델 모두 발전
        
- **활용**: 고품질 이미지 생성, 데이터 증강, 딥페이크 등.
    

![oaicite:23](https:)  
💡 이미지 4: 생성자와 판별자가 상호작용하며 학습하는 기본 GAN 구조입니다. ([GeeksforGeeks](https://www.geeksforgeeks.org/generative-adversarial-network-gan/?utm_source=chatgpt.com "Generative Adversarial Network (GAN) - GeeksforGeeks"))

---


[![Schematic diagram of bayesian optimization process | Download ...](https://tse1.mm.bing.net/th/id/OIP.iSXDKJYdbOqp7EK-F0OLJwHaHI?pid=Api)](https://www.researchgate.net/figure/Schematic-diagram-of-bayesian-optimization-process_fig3_370138057)

---

## 🧠 Bayesian Optimization (베이지안 최적화)

### 🔍 개요

- 평가 비용이 매우 큰 함수(예: 하이퍼파라미터 튜닝, 실험 설계)의 최적값을 찾기 위한 **샘플 효율적인 탐색 기법**입니다.
    
- 직접 함수를 여러 번 평가하기보다, **확률 모델(surrogate model)**을 사용해 다음 샘플 지점을 예측하고 평가하는 방식.
    

---

### 📈 주요 구성 요소

- **Surrogate Model**: 주로 Gaussian Process(GP)를 사용해 함수의 평균과 불확실성 영역을 추정
    
- **Acquisition Function**: Expected Improvement(EI), Upper Confidence Bound(UCB) 등  
    → surrogate가 기대하는 최적 포인트를 **탐색(explore)** 또는 **이용(exploit)** 하기 위한 기준 함수
    

---

### 🔁 반복 과정 흐름

1. 초기 샘플링 데이터 수집
    
2. Gaussian Process를 통해 함수 예측 및 불확실성 추정
    
3. Acquisition 함수 계산 → 최적 후보 지점 결정
    
4. 해당 지점 평가 → 결과 데이터에 추가
    
5. surrogate 모델 업데이트 → **이 과정을 반복해 최적값 탐색**
    

---

### 🧾 이미지 설명

위 이미지(`turn0image3`)는 전체 BO 프로세스를 요약한 **플로우차트 구조**입니다:

- GP 모델 업데이트 → Acquisition 함수 최적화 → 새로운 샘플 선택 → 반복
    
- 흐름의 반복 과정과 **exploit**과 **explore**의 균형 전략을 시각적으로 잘 표현하고 있습니다. ([sciencedirect.com](https://www.sciencedirect.com/science/article/pii/S2589004221007495?utm_source=chatgpt.com "Bayesian optimization for goal-oriented multi-objective inverse ..."), [nature.com](https://www.nature.com/articles/s41524-021-00662-x?utm_source=chatgpt.com "Bayesian optimization with adaptive surrogate models for automated ..."))
    

---

### ✅ 요약 정리

|항목|내용|
|---|---|
|**적합한 상황**|평가 비용이 비싼 함수의 최적화 (ex: 실험설계, 하이퍼파라미터 튜닝)|
|**주요 구성 요소**|Gaussian Process (surrogate), Acquisition Function (EI, UCB 등)|
|**장점**|적은 평가 횟수로도 전역 최적점을 찾을 수 있음|
|**단점**|모델 설정(커널, acquisition 함수) 민감, 계산 복잡도 있음|
|**대표적인 활용 사례**|머신러닝 하이퍼파라미터 튜닝, 물질 최적화, 로보틱스 실험 등|



---

## 📋 요약 비교표

|모델/기법|목적/분류|구조 요약|주요 활용|
|---|---|---|---|
|**XGBoost Regression**|지도학습 (회귀)|Boosted 트리 반복 학습|수치 예측, 피처 중요도 분석|
|**MLP**|지도학습 (분류/회귀)|완전 연결 다층 신경망|일반 예측 문제|
|**VAE**|생성 모델 (비지도)|Encoder–latent–Decoder 구조의 확률 모델|데이터 생성, 이상치 탐지|
|**GAN**|생성 모델 (비지도)|Generator vs Discriminator 적대 학습 구조|고품질 이미지 생성|
|**Bayesian Opt.**|최적화 알고리즘|Gaussian Process + Acquisition Function|하이퍼파라미터 최적화|

---

