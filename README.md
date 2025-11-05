# Jump-up-Labs-TEMPLATE
# 국립한밭대학교 인공지능소프트웨어학과 iSW팀

**팀 구성**
- 30242860 채희주
- 20221075 정현서
- 20221073 정은주
- 20221067 장채은
- 20231077 최은실

---

## Project Background
자율주행 기술의 급속한 발전과 상용화에 따라 도로 환경을 정확하게 인식하는 기술의 중요성이 꾸준히 커지고 있음
**교통 표지판 인식(Traffic Sign Recognition, TSR)** 은 자율주행 시스템과 첨단 운전자 보조 시스템(ADAS)의 핵심 인식 모듈로서 운전자에게 실시간 교통 표지 정보를 제공하거나 차량 제어 신호를 생성하는 역할을 함
그러나 실제 주행 환경에서는 야간, 비, 안개 등과 같은 악조건에서 인식 정확도가 종종 저하되는 문제가 발생함
이에 본 연구에서는 다양한 환경 조건에서도 안정적으로 작동하는 강인한 객체 탐지 모델의 필요성을 인식하고 YOLOv8 기반 교통 표지판 인식 모델의 성능 향상을 목표로 함

---

## Dataset
- 데이터셋 : Kaggle에서 공개된 교통 표지판 데이터셋 사용
- 전체 이미지 수 : 877장
- 클래스 수 : 4종
  - 속도 제한(Speed Limit) : 783장
  - 신호등(Traffic Light) : 170장
  - 횡단보도(Crosswalk) : 200장  
  - 정지(Stop) : 91장
  <img width="346" height="234" alt="image" src="https://github.com/user-attachments/assets/fc5caf95-4e25-4831-92ba-29b99318d944" />
  &rarr; 클래스 불균형


- 이미지 증강: Albumentations 라이브러리를 사용
  - 밝기, 대비, 블러, 안개, 비 등
- 데이터셋 분할 : 무작위로 셔플 후 &rarr; Train 80% / Validation 10% / Test 10%
- **Train : 원본 이미지 + 야간/악천후 증강 이미지**
  &rarr; 다양한 시나리오를 반영하기 위함
- **Validation / Test : 원본 이미지**
  &rarr; 모델의 일반화 성능을 공정하게 평가하기 위함
  
  <img width="818" height="160" alt="image" src="https://github.com/user-attachments/assets/491f7d8b-9676-45d5-9f12-127148788b91" />

---

## YOLOv8 (You Only Look Once)
- Backbone
 고급 CNN을 활용한 다중 스케일 특징 추출

- Neck
 서로 다른 스케일의 특징을 융합하여 작은 물체부터 큰 물체까지 탐지 성능을 향상

- Head
 **Anchor-Free 구조** : 앵커 박스 없이 직접 예측 &rarr; **모델 구조 단순화 및 유연성 향상**

---

## Binary Cross Entropy (BCE)
<img width="450" height="43" alt="image" src="https://github.com/user-attachments/assets/3e4ba742-afe3-4166-8c03-29ab063b4d4b" />

- 𝑦∈{0,1} : 실제 정답(Ground truth, 실제 레이블)
- 𝑦 ̂∈{0,1} : Sigmoid 출력값 (예측 확률)

- 이진 분류에서 사용되는 대표적인 손실 함수
- YOLOv8의 기본 손실 함수로 사용됨
- 예측 확률과 실제 레이블 간의 차이를 계산함
- **정답 클래스에 대해 모델이 과도하게 확신하도록 만드는 경향이 있음**
  &rarr; 클래스 불균형이나 분류가 어려운 샘플을 다룰 때 성능 저하를 유발할 수 있음

---

## Focal Loss
<img width="307" height="39" alt="image" src="https://github.com/user-attachments/assets/34fa5e81-8037-4913-bcc8-52195fef4cf2" />

- 𝑝_𝑡 : 실제 클래스에 대한 예측 확률
- 𝛼 : 클래스 가중치
- 𝛾 : Focusing parameter &rarr; 쉬운 샘플의 손실을 줄이고 어려운 샘플에 집중하도록 조정

- 손실의 가중치를 조정하여, 모델이 쉬운 샘플보다 어려운 샘플에 더 집중하도록 함
- α는 소수 클래스에 더 큰 가중치를 부여 
  &rarr; **클래스 불균형 문제 완화에 도움**
- γ는 오분류된 샘플에 더 큰 손실을 부여함

---

## Label Smoothing Cross Entropy Loss
<img width="219" height="57" alt="image" src="https://github.com/user-attachments/assets/b294a327-9c10-4528-8971-3b5b47f8701f" />

- 𝛼∈{0,1} : 스무딩 계수
- 𝐾 : 클래스의 개수
- 𝑦 ̃_𝑘 : 스무딩된 라벨 분포

<img width="186" height="266" alt="image" src="https://github.com/user-attachments/assets/15bf2b45-a321-4d71-94e1-1770f9207e92">

- 𝑦 ̃_𝑖  : 스무딩된 라벨 분포
- 𝑝_𝑖 : Softmax 출력값 (예측 확률)

- 실제 정답 라벨에 **작은 불확실성**을 추가하여 모델이 정답 클래스에 과도하게 확신하지 않도록 함
  **&rarr; 일반화 성능 향상 및 과적합 방지**
- ex ) [0, 0, 1, 0] &rarr; [0.05, 0.05, 0.85, 0.05] 

---

## Performance Metrics
- Precision : 모델이 탐지한 객체 중 실제로 올바른 객체의 비율
- Recall : 실제 객체 중 모델이 올바르게 탐지한 비율
- mAP@50 : 예측 박스와 실제 박스의 IoU가 50% 이상일 때 클래스별 평균 정밀도를 계산한 값
- mAP@50-95 : IoU 임계값을 50%에서 95%까지 5% 간격으로 변화시키며 계산한 mAP의 평균값
  
---

## Comparison of Loss Function
<img width="701" height="190" alt="image" src="https://github.com/user-attachments/assets/3261f99c-705a-4fd9-bb57-3355a16dda12" />

- BCE는 전반적으로 균형 잡힌 성능을 보여 baseline 모델로 작용함
- Focal Loss는 가장 높은 재현율(Recall, 0.870)을 기록하여 모델이 **놓치는 객체가 상대적으로 적었음**을 의미함
**&rarr; 희귀하거나 분류가 어려운 샘플에 대해 상대적으로 강인함을 시사함**
- Label Smoothing은 가장 높은 정밀도(Precision, 0.916)를 보였지만, 재현율(Recall, 0.785)은 가장 낮았음
  **불확실한 경우 예측을 회피하는 보수적인 경향을 보임**
**&rarr; 과신을 줄이고 일반화 성능을 향상시킴**

- 어떤 단일 손실 함수도 모든 지표에서 최적의 성능을 보이지 않음 
**&rarr; 따라서 Label Smoothing과 Focal Loss의 장점을 결합하여 새로운 손실 함수를 설계함**

---

## Combined Loss : Label Smoothing + Focal Loss
<img width="260" height="266" alt="image" src="https://github.com/user-attachments/assets/7fdb9a25-f93f-43f9-82b0-e62c0e6bfb54" />

- 𝑝_𝑖 : Softmax 출력값 (예측 확률)
- 𝑦 ̃_𝑖  : 스무딩된 라벨 분포
- 𝛾 : Focusing parameter &rarr; 쉬운 샘플의 손실을 줄이고 어려운 샘플에 집중하도록 함

Label Smoothing **(정답 클래스에 불확실성을 부여)** 과
Focal Loss **(어려운 샘플에 더 높은 가중치를 부여)** 를 결합하여
**&rarr; 과적합을 방지하고 어려운 샘플에 집중하도록 개선**

### Experiments to find the optimal hyperparameters 
<img width="711" height="270" alt="image" src="https://github.com/user-attachments/assets/666faecf-04cc-4c00-85f7-4cb0b0442b23" />

- **𝑦 ̃_𝑖=0.05,  𝛾=2.0** 일 때, 모델은 Precision과 mAP@50-95에서 약간의 우위를 보였으며 전반적으로 높은 성능을 달성함

### Fine-grained hyperparameter tuning focused on 𝑦 ̃_𝑖 
<img width="709" height="155" alt="image" src="https://github.com/user-attachments/assets/2e25e62b-b6c3-4ae1-bdde-6a1f405e3170" />

- When **𝑦 ̃_𝑖=0.05,  𝛾=2.0** 일 때, 모델은 Precision, mAP@50, mAP@50-95에서 최고 성능을 보이면서도 높은 Recall을 유지함 
- 전반적인 성능 측면에서 가장 최적의 설정으로 확인
  
### Additional tuning of Confidence & IoU thresholds to improve post-processing performance
<img width="877" height="127" alt="image" src="https://github.com/user-attachments/assets/05d1aba5-af44-47bb-bf61-e9a4f75da34b" />

- Confidence threshold = 0.25일 때, 모델은 Precision(0.969), Recall(0.885), mAP@50(0.939), mAP@50-95(0.822)로 최고 성능을 기록함 
- 최종 하이퍼파라미터 설정 : **𝑦 ̃_𝑖=0.05,  𝛾=2.0,  conf=0.25**

---

## Conclusion
<img width="394" height="231" alt="image" src="https://github.com/user-attachments/assets/63df1071-a7fe-4528-9717-4b2086bded78" />

- 테스트 데이터셋을 활용하여 교통 표지판을 정확하게 탐지하고 분류함
- 결합 손실 함수를 적용한 YOLOv8 모델이 최고의 성능을 달성함 : **Precision: 0.969, Recall: 0.885, mAP@50: 0.939, mAP@50–95: 0.822**
- **야간 및 악천후 환경에서도 정확하고 안정적인 객체 탐지 성능을 보여줌**
- 향후 연구에서는 실시간 탐지 및 경량화 아키텍처 구현을 통해 실제 응용으로 확장할 계획 

## Project Outcome
- 국제 학술대회 ICTC2025 제출
