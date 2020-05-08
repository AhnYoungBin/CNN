## YOLO(You Only Look Once)
Unified, Real-Time Object Detection

### Abstract
object dtection을 공간적으로 분리된 bounding boxes와 클래스 확률로 regression을 간주함.

single-neural network는 한 번의 평가로 전체 이미지에서 bounding boxes와 클래스 확률을 예측함.

전체 detection pipeline이 single-neural network이기 때문에, detection 성능에 대해 직접적으로 end-to-end로 최적화될 수 있으며 이러한 구조는 매우 빠르게 진행 됨.

YOLO는 localization error를 더 많이 만들지만, 배경에 대한 false positive으로 예측 가능성은 낮음. 

결과적으로 객체의 일반적인 표현을 학습한다고 말할 수 있음.

사물의 일반적 표현을 다른 도메인으로 일반화할 때 DPM(deformable parts models)과 R-CNN을 포함한 다른 탐지 방법들 능가함.

### Introduction
현재의 detection system은 분류기에 detection 역할을 하도록 재설정하는 방식으로 객체를 탐지하기 위해서, 시스템은 객체를 분류기가 받고 테스트 이미지 내에서 다양한 위치와 크기를 평가함.

DPM(deformable parts models)같은 경우는 sliding window를 사용하여 분류기가 전체 이미지에 대해 모든 공간에서 균등하게 실행되도록 함.
R-CNN의 경우는 region proposal의 방법을 사용하여 먼저 이미지 내에 잠재적인 bounding boxes를 생성하고 그 제안된 boxes에 대해 분류기를 실행하여 분류 이후 bounding boxes를 정교하게 하고, 중복되는 detection을 제거하여 다른 객체에 기반해서 boxes를 재점수함. - 이러한 방식은 복잡한 파이프라인은 개개인의 요소가 분리되어서 학습되어야 하기 때문에 연산량이 많고 최적화 하는데 어려움.

object detection을 단일 regression 문제로 재구성하고, 이미지 픽셀에서 bounding box 좌표와 클래스 확률까지 쭉 이어지도록 하였다. - 그래서 YOLO(You Only Look Once)라고 명칭함. - "객체가 무엇이고 어디 있는지를 예측하기 위해 이미지를 '단 한 번만 본다.'"

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/81404373-477d9a80-9170-11ea-9f92-b1d6d10bd90f.png" width="75%"></p>

The YOLO Detection System.

    - 1. 먼저 입력이미지를 448 x 448의 크기로 조정한다.
    - 2. 이미지에 대해 단일 Convolution Network를 실행함.
    - 3. NMS(Non-Max Suppresion - 중복된 detection을 제거) - 모델의 신뢰도에 의한 결과 detection의 임계값을 정함.
  
간단하게 말하자면 단일의 Convolutional Network는 동시에 복수의 Bounding boxes를 예측하고, boxes에 대한 클래스 확률을 예측함.
따라서 전체 이미지에 대해 학습하고 detection 성능을 직접적으로 최적화함.

YOLO의 장점 3가지.

1. 매우 빠름.

detection을 regression 문제로 재구성하여 복잡한 파이프라인이 필요하지 않게 됨.

Titan X GPU에서 배치 없이 45fps(frame per second), 빠른 버전 150fps보다 빠름. - real time으로 적용할 수 있다는 것을 의미, 다른 real time 시스템에 비해 두 배가 넘는 mAP를 달성함.

2. 이미지에 대해 전체적으로 추론함.

Sliding window와 region proposal 기반의 기술과 다르게, YOLO은 학습과 테스트시에 전체 이미지를 보기 때문에, 명백하게 그들의 외관과 같은 클래스에 대한 문맥상의 정보를 encodes한다.

YOLO는 Fast R-CNN과 비교해서, 절반 이하의 배경 오류를 발생 시킴.

3. 객체의 일반적인 표현을 학습함.

자연 이미지와 예술 이미지 학습하였을 때, YOLO는 DPM과 R-CNN보다 높은 성능을 얻을 수 있었음.
또한 매우 일반적이여서 새로운 도메인이나 입력에 대해서도 실패할 가능성이 적음.

### Unified Detection
전체 이미지로부터의 특성으로 각 bounding box를 모든 클래스에 대해 동시에 예측함. - 모든 객체를 전체적으로 잘 추론하고 있다는 것을 의미.

따라서 높은 AP(Average Precision)을 유지하며 end-to-end로 학습되고 real time속도를 가능하게 함.

동작 방식으로는 입력 이미지를 S x S grid로 나눠서 객체의 중심에 grid cell이 들어갔다면, 그 객체를 탐지하는 역할을 함.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/81413953-3f7a2680-9181-11ea-9f98-bffa77d36696.png" width="75%"></p>

각 grid cell은 bounding boxes B와 boxes에 대한 신뢰 점수를 예측함. - 신뢰 점수는 box가 객체를 보유하고 있다고 생각하는 모델의 신뢰도와 예측하는 box의 정확도를 반영, <img width="124" alt="스크린샷 2020-05-08 23 11 54" src="https://user-images.githubusercontent.com/45933225/81414002-4f920600-9181-11ea-8101-e8cc9f7af219.png"> 으로 표현.
.

각 bounding box는 5개의 예측으로 구성됨. - x, y, w, h 그리고 신뢰도이다.
        
        - (x, y)는 grid cell의 경계를 기준으로 box의 중심좌표를 나타냄.
        - 너비와 높이는 전체 이미지에 대해 예측함.
        - 신뢰도 예측은 예측된 box와 어느 ground truth box 사이의 IOU를 나타냄.
        
각 grid cell은 조건부 클래스 확률인 C를 <img width="102" alt="스크린샷 2020-05-08 23 12 12" src="https://user-images.githubusercontent.com/45933225/81414045-5ae53180-9181-11ea-99e0-0f49f2057144.png">
 예측함. - 여기서 확률은 객체를 보유하고 있는 grid cell에 대한 조건부를 나타냄.
boxes의 갯수인 B에 상관하지 않고, 오직 grid cell당 하나의 클래스 확률을 예측함. - 테스트시에는 조건부 확률과 개개인의 신뢰도 예측을 곱하면 위에 합쳐져 있는 식이 나옴.

결과적으로 각 box에 대해 클래스별 신뢰도를 알려줌. - 점수는 box에서 클래스가 나타나는 확률과 예측된 box가 객체와 얼마나 잘 맞는지를 담고 있다.




