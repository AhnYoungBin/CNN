## kaggle_dogs-vs-cats
본 데이터는 Kaggle 공개 데이터를 이용함.

이미지에 개 또는 고양이가 포함되어 있는지 여부를 분류하는 알고리즘으로 이것을 사람이 구별하기는 쉬울 수 있지만, 컴퓨터에서는 조금 어려울 것임.

### 데이터 정보

<p align="center">Dogs-vs-Cats Class</p>
<p align="center"><img src="https://user-images.githubusercontent.com/45933225/83288445-51019c00-a21e-11ea-8556-b35142464dc6.png" width="50%"></p>

    데이터 포맷 : jpg, jpeg
    총 25,000장, Train : 17,000장, Validation : 4,000장, Test : 4,000장
    해상도 : 고유 이미지 사이즈가 다름.
    class : Dog, Cat

다음 아래와 같이 전체적인 데이터를 한 눈에 확인할 수 있음.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/83288474-5f4fb800-a21e-11ea-8219-ad1218bcd73b.png" width="75%"></p>

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

VggNet은 2014년 ILSVRC에서 2위를 차지했지만 훨씬 단순한 구조로 이해와 변형이 용이하다는 장점이 있다.

특징으로는 VGGNet은 작은 필터 크기의 컨볼루션 연산이 있으며, 단점으로는 많은 파라미터가 존재한다는 것을 알 수 있다.
파라미터가 많은 것은 Gradient Vanishing, Over Fitting 등의 문제가 발생할 가능성이 높다는 것을 의미한다.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74590059-48f66580-504e-11ea-9952-0828a186eb60.png" width="75%"></p>

VggNet 16Layer의 기본 구성을 위 그림에서 확인할 수 있다. 실험에서 차이점으로는 각 블록에 Kernel_initializeer = 'he_normal'을 추가하였으며 Vanishing/Exploding을 완하하기 위해 Convolution과 Max Polling 사이에 Batch Normalization을 추가 하였다. 그리고 출력층에 대해서 Multi Class가 아니여서 softmax가 아닌 binary class에 적합한 sigmoid함수를 사용하였다.

모델 컴파일의 경우 최적화 방법은 SGD(learning rate = 0.001, momentum = 0.9)함수를 사용했으며 손실 함수는 binary_crossentropy로 설정하였다. 그리고 하이퍼 파라미터는 (train, validation)batch_size 20, image_size 224 x 224, epoch 250, (train, validation)step 80, 20으로 설정하였다.

실험 결과로는 Accuracy 95.05%, Loss 0.1351이 나오는 것을 확인 하였다.

- feature map

##### accuracy 95.05%, loss 13.51% feature map

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74591625-d345c600-505c-11ea-8f0f-56a6d300223e.png" width="75%"></p>

##### accuracy 49.45%, loss 69.32% feature map

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74591647-038d6480-505d-11ea-96e9-5193de6d5b6f.png" width="75%"></p>

높은 정확도와 낮은 정확도의 특징 맵을 비교하였다.
정확히 육안으로 보이지는 않지만 대략적인 판단으로는 높은 정확도는 이미지의 엣지를 잘 찾아가는 것 같고, 정확도가 낮은 것은 잘 찾지 못하거나 안보이는 것이 더 많은 것을 확인할 수 있었다.

#### 2. ResNet(Layer - 50, 101 Layer)

#### 3. DenseNet(Pooling - Max, Avg)

#### 4. EfficientNet(Optimizer - SGD, Adam)
