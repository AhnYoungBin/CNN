## kaggle_dogs-vs-cats
본 데이터는 Kaggle 공개 데이터를 이용함.

이미지에 개 또는 고양이가 포함되어 있는지 여부를 분류하는 알고리즘으로 이것을 사람이 구별하기는 쉬울 수 있지만, 컴퓨터에서는 조금 어려울 것임.

### 데이터 정보

    데이터 포맷 : jpg, jpeg
    총 25,000장, Train : 17,000장, Validation : 4,000장, Test : 4,000장
    해상도 : 고유 이미지 사이즈가 다름.
    class : Dog, Cat

다음 아래와 같이 전체적인 데이터를 한 눈에 확인할 수 있음.

#### 데이터 보완
데이터 레이블을 변경하지 않고 픽셀 변화시키는 CNN의 성능 향상을 위하여 Tensorflow에서 제공해주는 Data Augmentation을 사용함.

다음과 같이 데이터에 적용함.

    Train data : rotation_range = 40, shear_range = 0.2, zoom_range = 0.2, horizontal_flip = True, rescale = 1./255
    Validation data : rescale = 1./255
    Test data : rescale = 1./255

### 실험 소개 및 수행 내역
단순한 모델 학습이 아닌 각 모델마다 비교하고자 하는 것을 하나 정하여서 하고자 함.

그래서 다음과 아래와 같이 순서대로 비교하였음.

    - 1. VggNet(Accuracy - Featrue Map)
    - 2. ResNet(Layer - 50, 101 Layer)
    - 3. DenseNet(Pooling - Max, Avg)
    - 4. EfficientNet(Optimizer - SGD, Adam)

#### 1. VggNet(Accuracy - Featrue Map) 

#### 2. ResNet(Layer - 50, 101 Layer)

#### 3. DenseNet(Pooling - Max, Avg)

#### 4. EfficientNet(Optimizer - SGD, Adam)
