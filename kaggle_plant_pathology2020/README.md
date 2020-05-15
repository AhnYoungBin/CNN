## kaggle_plant_pathology2020
- Google에서 제공해주는 Colab gpu를 사용하여 다음과 같이 실험을 진행하였음.

- 이 대회는 Computer Vision and Pattern Recognition Conference(CVPR 2020)에서 Fine-Grained Visual Categorization FGVC7 워크숍 일부로써 kaggle Web Site를 통하여 데이터를 제공 받음.

#### 문제
=> 농작물에 영향을 미치는 많은 질병의 오진은 화학 물질의 오용으로 이어질 수 있으며, 내성 병원균 균주의 출현, 투입 비용 증가, 심각한 경제적 손실 및 환경 영향으로 더 많은 발생이 이어질 수 있다. 따라서 컴퓨터 비전 기반 모델로 감염된 조직의 나이, 유전적 변이 및 나무 내의 빛 상태로 인한 증상의 큰 변화 등을 알고자 함.

#### 목표 
사과 잎의 사진이 주어지면 건강 상태를 정확하게 평가할 수 있을까?

=> 이 대회는 건강에 좋은 잎, 사과 녹에 감염된 잎, 사과 녹병이 있는 잎 및 둘 이상의 질병을 가진 잎을 구별하는데 의미를 둠.

    1. 많은 질병을 정확하게 구별하고, 한 잎에서 하나 이상의 질병을 정확하게 구별.
    2. 희귀한 질병과 새로운 증상을 다룸.
    3. 깊이 인식 - 각도, 빛, 그늘, 잎의 생리적 연령에 대응.
    4. 학습 중 관련 기능 검색하기 위한 식별, 주석, 수량화 및 컴퓨터 비전 안내에 전문적인 지식을 통합.

#### 데이터 정보

<p align="center">Plant Pathology Class</p>
<p align="center"><img src="https://user-images.githubusercontent.com/45933225/82079740-4fb07980-971e-11ea-9c9e-c5e74ac4709e.png" width="100%"></p>

    데이터 포맷 : jpg, jpeg
    총 3642장, Train : 1821장 Test : 1821장(Test data label 없음).
    해상도 : 2048 * 1365
    class : healthy(건강), multiple_diseases(복합 질병),rust(녹병), scab(붉은곰팡이병)
    
    
- Train data graph

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/81252299-8fad9780-9060-11ea-8c17-99b1abd089ef.png" width="75%"></p>

=> combinations 516개, healthy 91개, rust 622개, scab 592개 총 Train data 1821개 확인.

#### 데이터 분류

    1. Train data 각 클래스 별로 8:2 비율로 나누어서 validation data 생성.   -  모델의 성능을 일반화하기 위해서 validation data을 이용(일반적으로 8:2 비율이 적절)
    2. 교차 검증(Cross Validation) KFold 적용.    -   모델이 데이터를 판단하기 위한 데이터 수가 적은 경우 검증 성능의 신뢰도가 떨어지는 것을 해결하고자 함(반대로 정확도 향상을 위한것으로도 말할 수 있음)
    
#### 데이터 보완
데이터의 수가 부족하여 데이터를 보완하느 방법으로 이미지의 레이블을 변경하지 않고 픽셀 변화 시키는 CNN의 과적합 방지와 성능 향상을 위한 Data Augmentation의 albumentations Moudle을 사용하였음.

다음과 같이 데이터에 적용함.

    Train data : Resize, RandomCrop, Resize, Flip, ShiftScaleRotate, HorizontalFlip
    Validation data : Resize
    Test data : Resize
    
#### 실험 소개 및 수행 내역
문제 해결 방법으로 Computer Vision(CV)의 분야에서 사용하는 Convolution Neural Network(CNN)을 이용하여 컴퓨터가 영상(이미지 및 비디오) 디지털 이미지의 특징을 학습하여 특정 패턴을 사용한 이미지를 분류하고자 함.

다음과 같이 절차를 밟아서 수행함.

    - 1. 데이터에 적절한 모델 선정(VggNet, ResNet, DenseNet, EfficientNet).
    - 2. 데이터에 적절한 모델 버전 및 하이퍼파라미터 최적화(Model(v1, v2 etc..) ,Hyper parmeter(Image Resolution, optimizer, learning rate etc..))
    - 3. 정확도 향상을 위한 기술 적용(K-Fold, Ensemble, Attention).

##### 1. 데이터에 적절한 모델 선정
대표적으로 Convolution Neural Network에서 사용하는 모델을 선택하여 실험하였으며 모델을 제외한 나머지 조건들을 동일하게 설정하여 비교하고자 하였음.

모델은 다음과 같이 사용함.

    - VggNet, ResNet, DenseNet, EfficientNet

하이어파라미터 초기화

      - image size(height, width, channel) : 342, 512, 3
      - epoch : 150
      - batch size(train, validation, test) : 8, 8, 1
      - step(train, validation, test) : 80, 20, 1
      - label : one-hot encoding    -   using Categorical entropy
      - optimizer : Adam(learning rate : 0.01)  -   validation loss에 맞게 laerning rate 조정하였음.

- VggNet

kaggle 제출 결과 93.7 Accuracy 얻을 수 있었음.

Train, Validation data 전체를 이용하여 학습 Test 결과 accuracy : 0.95442, loss : 0.14095 확인함과 동시에 어떤 클래스를 분류를 잘 못하는지 확인하고자 Confusion Matrix를 이용하였음.  

Confusion Matrix 

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/82085133-7b842d00-9727-11ea-9f40-c18a3fddccaf.png" width="35%"></p>

multi diseases, scab, healthy, rust 순서대로 잘못 판단하고 있었으며 일단 데이터의 수가 너무 적은 multi diseases을 판단하기 어려울 것이라고 예상함 그리고 복합적인 질병을 다른 단일 병들과 구분 짓기 어려울것이라고 생각이 듬.

- ResNet

kaggle 제출 결과 56.3 Accuracy 얻을 수 있었음.

Train, Validation data 전체를 이용하여 학습 Test 결과 accuracy : 0.67326, loss : 0.61089 확인함과 동시에 어떤 클래스를 분류를 잘 못하는지 확인하고자 Confusion Matrix를 이용하였음.

Confusion Matrix

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/82089570-d0777180-972e-11ea-901d-45d71243c07e.png" width="35%"></p>

multi diseases, healthy, scab, rust 순서대로 잘못 판단하고 있었으며 multi diseases 위와 같은 이유로 똑같이 분류를 잘 못할거라고 생각하였으며 healthy는 주로 scab으로 많은 양의 데이터를 분류하고 있음을 확인할 수 있었다.

- DenseNet

위의 하이퍼파라미터 같은 조건으로 실행한 결과 모델 학습이 원할하게 진행되지 않아서 따로 기록을 하지 않았음.

- EfficientNet

kaggle 제출 결과 94.6 Accuracy 얻을 수 있었음.

Train, Validation data 전체를 이용하여 학습 Test 결과 accuracy : 0.1600, loss : 0.94393 확인함과 동시에 어떤 클래스를 분류를 잘 못하는지 확인하고자 Confusion Matrix를 이용하였음.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/82090450-93ac7a00-9730-11ea-9958-392bf93c8b76.png" width="35%"></p>

multi diseases, scab, healthy, rust 순서대로 잘못 판단하고 있었으며 다른 모델들보다 좋은 성능을 얻을 수 있었음.

그래서 전체적으로 학습한 그래프는 다음과 같음.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/82093860-e38e3f80-9736-11ea-99a8-41147d15090b.png" width="50%"></p>

바로 위에서도 말했듯이 모든 면에서 EfficientNet이 좋은 성능을 얻을 수 있다는 것을 한눈에 확인할 수 있었음.

##### 2. 데이터에 적절한 모델 버전 및 하이퍼파라미터 최적화
