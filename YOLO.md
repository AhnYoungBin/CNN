## YOLO(You Only Look Once)
Unified, Real-Time Object Detection

### Abstract
object dtection을 공간적으로 분리된 bounding boxes와 클래스 확률로 regression을 간주함.

single-neural network는 한 번의 평가로 전체 이미지에서 bounding boxes와 클래스 확률을 예측함.

전체 detection pipeline이 single-neural network이기 때문에, detection 성능에 대해 직접적으로 end-to-end로 최적화될 수 있으며 이러한 구조는 매우 빠르게 진행 됨.

YOLO는 localization error를 더 많이 만들지만, 배경에 대한 false positive으로 예측 가능성은 낮음. 

결과적으로 객체의 일반적인 표현을 학습한다고 말할 수 있음.

사물의 일반적 표현을 다른 도메인으로 일반화할 때 DPM과 R-CNN을 포함한 다른 탐지 방법들 능가함.

### Introduction
