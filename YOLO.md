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

<img width="335" alt="스크린샷 2020-05-08 21 09 59" src="https://user-images.githubusercontent.com/45933225/81404373-477d9a80-9170-11ea-9f92-b1d6d10bd90f.png">

The YOLO Detection System.
