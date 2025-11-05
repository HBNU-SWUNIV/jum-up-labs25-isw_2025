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
자율주행 기술의 급속한 발전과 상용화에 따라 도로 환경을 정확하게 인식하는 기술의 중요성이 꾸준히 커지고 있다.
**교통 표지판 인식(Traffic Sign Recognition, TSR)** 은 자율주행 시스템과 첨단 운전자 보조 시스템(ADAS)의 핵심 인식 모듈로서 운전자에게 실시간 교통 표지 정보를 제공하거나 차량 제어 신호를 생성하는 역할을 한다.
그러나 실제 주행 환경에서는 야간, 비, 안개 등과 같은 악조건에서 인식 정확도가 종종 저하되는 문제가 발생한다.
이에 본 연구에서는 다양한 환경 조건에서도 안정적으로 작동하는 강인한 객체 탐지 모델의 필요성을 인식하고 YOLOv8 기반 교통 표지판 인식 모델의 성능 향상을 목표로 한다.

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
ex ) [0, 0, 1, 0] &rarr; [0.05, 0.05, 0.85, 0.05] 

---

## Performance Metrics
- Precision : The proportion of detected objects that are actually correct
- Recall : The proportion of true objects that are correctly detected by the model
- mAP@50 : Average precision per class when predicted boxes overlap with ground-truth boxes by ≥ 50%
- mAP@50-95 : Average mAP over IoU thresholds from 50% to 95% in 5% increments

---

## Comparison of Loss Function
<img width="701" height="190" alt="image" src="https://github.com/user-attachments/assets/3261f99c-705a-4fd9-bb57-3355a16dda12" />

- BCE showed overall balanced performance, serving as a stable baseline model.
- Focal Loss achieved the highest Recall (0.870), indicating that the model **missed fewer objects** overall.
**&rarr; Suggests relative robustness to hard or rare samples**
- Label Smoothing showed the highest Precision (0.916) but the lowest Recall (0.785), indicating **a tendency for the model to avoid predictions in uncertain cases.**
**&rarr; more conservative, reduces overconfidence, improves generalization**

- No single loss function performs best across all metrics 
**&rarr; Combines the strengths of Label Smoothing and Focal Loss**

---

## Combined Loss : Label Smoothing + Focal Loss
<img width="260" height="266" alt="image" src="https://github.com/user-attachments/assets/7fdb9a25-f93f-43f9-82b0-e62c0e6bfb54" />

- 𝑝_𝑖 : Softmax output (predicted probability)
- 𝑦 ̃_𝑖  : Smoothed label distribution
- 𝛾 : Focusing parameter → reduces loss for easy samples, focuses on hard sample

Combines Label Smoothing **(adds uncertainty to true class)**
Focal Loss **(assigns higher weights to hard samples)**
**&rarr; Prevents overfitting and focuses on hard samples**

### Experiments to find the optimal hyperparameters 
<img width="711" height="270" alt="image" src="https://github.com/user-attachments/assets/666faecf-04cc-4c00-85f7-4cb0b0442b23" />

- When **𝑦 ̃_𝑖=0.05,  𝛾=2.0**, the model showed a slight advantage in Precision and mAP@50-95, and achieved overall high performance. 

### Fine-grained hyperparameter tuning focused on 𝑦 ̃_𝑖 
<img width="709" height="155" alt="image" src="https://github.com/user-attachments/assets/2e25e62b-b6c3-4ae1-bdde-6a1f405e3170" />

- When **𝑦 ̃_𝑖=0.05,  𝛾=2.0**, the model achieved the best performance in Precision, mAP@50, and mAP@50-95, while maintaining a high level of Recall. 
- Confirmed as the most optimal setting for overall performance

### Additional tuning of Confidence & IoU thresholds to improve post-processing performance
<img width="877" height="127" alt="image" src="https://github.com/user-attachments/assets/05d1aba5-af44-47bb-bf61-e9a4f75da34b" />

- When the confidence threshold of 0.25, the model achieved the best performance, recording the highest Precision (0.969), Recall (0.885), mAP@50 (0.939), and mAP@50-95 (0.822). 
- Final hyperparameter settings : **𝑦 ̃_𝑖=0.05,  𝛾=2.0,  conf=0.25**

---

## Conclusion
<img width="394" height="231" alt="image" src="https://github.com/user-attachments/assets/63df1071-a7fe-4528-9717-4b2086bded78" />

- Accurately detects and classifies traffic signs using the test dataset.
- YOLOv8 with combined loss function achieved best performance: **Precision: 0.969, Recall: 0.885, mAP@50: 0.939, mAP@50–95: 0.822**
- Demonstrates **accurate and stable object detection under nighttime and adverse weather.**
- For future work, we plan to extend the model to real-world applications by implementing real-time detection and lightweight architectures. 

## Project Outcome
- 국제 학술대회 ICTC2025 제출
