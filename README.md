# 기업 파산여부 예측(Classification)
### 📖 요약

- 데이터셋은 **6819개의 기업**들의 **파산 여부를 포함한 재무 정보 데이터**로 이루어졌다. (6819 x 96)
- 전통적인 지도학습 분류 모델 중 하나인 **SVM(Support Vector Machine) 모델**을 사용하여 **이진분류**를 실시한다.
- 학습 데이터의 **feature는 96개**로, 해석 시 이해하기 쉬운 2차원의 데이터로 만들기 위해 **PCA(Principal Component Analysis)** 및 **일변량 통계기반 선택**기법을 사용하여 **차원축소**를 진행한다.
- 데이터의 **클래스 불균형 문제(class imbalance)**로 편향된 모델을 수정하기 위해 **Over Sampling**  이후 학습 및 예측을 다시 실시하여 결과를 비교해본다.

---

### Dataset

<img width="686" alt="bankruptcy_data" src="https://github.com/HaseongJung/Bankrunptcy_Classification/assets/107913513/da3b1752-9112-4acb-bc7a-51cad6e50fe6">

### PCA

데이터들의 Scale이 학습에 영향을 주지 않도록 **Standard Scaler**를 통해 **정규화한 후,** Scikit-learn을 이용해  **PCA**를 진행하였다.

**PC1**, **PC2**을 추출한 결과 각각의 **`설명력이 약 0.13, 0.07`**로 원래 데이터의 **20%**밖에 설명하지 못한다.

따라서 PCA를 통해 추출한 변수들을 통해 예측하는 것이 바람직하지 않다.

### 일변량 통계기반 선택

독립변수와 종속변수사이에 통계적 관계가 존재하는지 일변량 통계량에 기초하여 판단하고 유의미한 독립변수들을 선별하는 **일변량 통계기반 선택기법**을 사용한다. (Scikt-learn SelectKBest모듈)

일변량 통계기반 선택 결과 종속변수인 파산 여부와 상관관계가 높은 2개의 변수는 `['ROA(A) before interest and % after tax', ' Net Income to Total Assets]` 로 추출되었다.

### 예측

위에서 선택한 변수들을 독립변수로 설정, 파산여부를 종속변수로 설정하고 학습/테스트 데이터의 비율은 8:2로 설정하여 예측을 진행한다.

결과는 다음과 같다.

<img width="285" alt="result1-1" src="https://github.com/HaseongJung/Bankrunptcy_Classification/assets/107913513/0bd0cacb-f9ad-4887-ba52-98fe005abc67">

0: 파산X, 1: 파산O

<img width="401" alt="result1-2" src="https://github.com/HaseongJung/Bankrunptcy_Classification/assets/107913513/f22dda9e-cbaf-43cf-92a9-017fb62a652c">

accuracy만 보면 모델의 성능이 좋아 보이나, **1에 대한 recall값과 f1-score가 현저히 낮은 것**으로 보아 모델이 **편향적 예측을 하는 것으로 보인다.**

이는 기업의 파산여부를 예측하는 목적에 매우 적합하지 않다고 생각된다.

원인은 파산한 기업(y=1)보다 파산하지 않은 기업(y=0)의 데이터가 월등히 많기 때문이라고 추측된다.

### Calss imbalance 문제 해결 (Over Sampling)

앞서 말했던 데이터의 **Class imbalance** 문제를 해결하기 위하여 **Under-sampling**과 **Over-sampling** 중 **데이터의 손실이 낮은 Over-sampling**기법을 사용하여 해결해보려고 한다. 

<img width="508" alt="over_sampling" src="https://github.com/HaseongJung/Bankrunptcy_Classification/assets/107913513/5d8951d4-836f-422c-ae88-2a67e1f37c81">

SMOTE 알고리즘을 이용하여 Over-sampling한 결과 **Class의 비율을 1:1**로 맞추어 주었다.

이후 class imbalance문제를 해결한 데이터로 다시 학습 및 테스트 결과는 다음과 같다.

<img width="284" alt="result2-1" src="https://github.com/HaseongJung/Bankrunptcy_Classification/assets/107913513/3c3b5cdb-64a7-417d-b3c3-623cb23b056f">

0: 파산X, 1: 파산O

<img width="392" alt="result2-2" src="https://github.com/HaseongJung/Bankrunptcy_Classification/assets/107913513/ecd7217d-251f-4fcd-a83d-aebb926c6dfb">

- 1에 대한 precision: `0.67 → 0.15`
- 1에 대한 recall: `0.08 → 0.82`
- 1에 대한 f1-score: `0.14 → 0.26`

class imbance문제 해결 결과 precsion(정밀도)은 감소, recall(재현율)은 상승하였고 f1-score는 소폭 상승하였다.

기업의 파산여부를 예측해 risk를 대비하는 이 프로젝트의 목적에는, **후자의 모델이 더 적합**하다고 판단되어진다. 완벽하지는 않지만 f1-score의 결과, 성능이 소폭 상승되었다.
