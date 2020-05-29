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

VggNet 16Layer의 기본 구성을 위 그림에서 확인할 수 있다. 실험에서 차이점으로는 각 블록에 Kernel_initializeer = 'he_normal'을 추가하였으며Vanishing/Exploding을 완하하기 위해 Convolution과 Max Polling 사이에 Batch Normalization을 추가 하였다. 그리고 출력층에 대해서 Multi Class가 아니여서 softmax가 아닌 binary class에 적합한 sigmoid함수를 사용하였다.

모델 컴파일의 경우 최적화 방법은 SGD(learning rate = 0.001, momentum = 0.9)함수를 사용했으며 손실 함수는 binary_crossentropy로 설정하였다. 그리고 하이퍼 파라미터는 (train, validation)batch_size 20, image_size 224 x 224, epoch 250, (train, validation)step 80, 20으로 설정하였다.

실험 결과로는 Accuracy 95.05%, Loss 0.1351이 나오는 것을 확인 하였다.

- feature map

##### accuracy 95.05%, loss 13.51% feature map

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74591625-d345c600-505c-11ea-8f0f-56a6d300223e.png" width="75%"></p>

##### accuracy 49.45%, loss 69.32% feature map

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74591647-038d6480-505d-11ea-96e9-5193de6d5b6f.png" width="75%"></p>

높은 정확도는 이미지의 특징을 찾아서 점점 특정 부분에 찾아가며 뚜렷하지만 낮은 정확도를 기록한 것을 보면 전체적으로 스무딩한 이미지를 볼 수 있으며 가끔 어떤 층에서는 특징을 못찾은 것도 볼 수 있었다.

#### 2. ResNet(Layer - 50, 101 Layer)

ResNet은 2015년 ILSVRC 대회에서 우승한 모델로써 신경망의 구조가 깊어짐에 따라 정확도가 저하되는 문제를 해결하기 위한 방법으로「잔류 학습」을 도입하였다. 간단히 말해, 기존 네트워크를 H(x)(x는 레이어의 입력) F(x) = H(x) - x 라고 하면 네트워크 F(x) + x가 대략 H(x)로 학습되도록 하는 것이다. 즉, 레이어의 입력과 출력 간의 차이를 학습 하고 저하되는 문제를 해결할 수 있었다. 이것은 왼쪽 아래에 있는 그림을 보면 확인할 수 있다. 오른쪽은 그림은 50개 이상 Layer에서 Bottleneck을 이용하는 구조로 건너 뛰는 것을 볼 수 있다. 이는 층이 깊어질수록 연산량이 늘어남에 따라 제안된 방법으로 채널 수가 1 x 1 Conv로 줄어드 다음 3 x 3 Conv를 거쳐 1 x 1 Conv를 다시 통과하여 채널 수를 다시 복구하는 것을 볼 수 있다.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74638207-cdf89080-51ae-11ea-93fc-4f4646158be5.png" width="75%"></p>

다음 아래 그림은 ResNet 논문에서 제안한 5개의 구조다. 다만 실험에서는 101Layer, 50Layer을 사용하였다. 

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74638228-dbae1600-51ae-11ea-8f6d-ba4e685cf445.png" width="75%"></p>

- 50, 101 Layer

Hyper parameter 초기화는 (train, validation)batch_size 20, epoch 550, Image size 224 x 224 추가적으로 Data Augmentation을 줬으며. 
(train, validation)step은 80, 20으로 설정하였다.

학습 결과 101 Layer 경우 Accuracy 92.70%, Loss 0.1951가 나왔으며, 50 Layer Accuracy 93.80%, Loss 0.1692가 나오는 것을 볼 수 있었으며 확연한 차이는 없지만 이 데이터에서는 50 Layer가 더 적합하다고 볼 수 있었으며 레이어층이 무조건 깊다고 좋은 학습 결과를 받을 수 있는 것은 아니었다.

#### - feature map(Conv_2D Layer)
다음은 입력층 부터 출력층까지 Convolution Layer층의 feature map을 보여준다.

##### 50-layer

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/75012532-82532900-54c5-11ea-93c4-2ca3511520c0.png" width="75%"></p>

##### 101-layer

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74638370-1ca62a80-51af-11ea-9a4c-d18b593d081a.png" width="75%"></p>
<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74638389-2465cf00-51af-11ea-8884-f7839e414b94.png" width="75%"></p>

비교적 짧은 50 Layer에서도 충분히 이미지에 특징을 찾아가는 것을 확인이 가능하다.

#### 3. DenseNet(Pooling - Max, Avg)

DenseNEt은 각 Layer와 후속 Layer 사이에 각각 하나씩 직접 L(L+1) 연결을 가지고 있다. 각 Layer는 모든 이전 Layer의 feature map을 입력으로 사용한다. 이 구성은 소멸 단계 문제를 완하하고, 전파를 강화하며, 재사용에 용이하며, 매개변수 수를 크게 감소시킨다.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/75467606-44776880-59cf-11ea-987c-26462f352d1a.png" width="75%"></p>

이전 ResNet에서도 유사한 방법이 사용되었다.
하지만 ResNet은 feature map을 추가하는 방식인 반면 DenseNet은 각각의 feature map을 연결하기 위한 구성을 지니고 있다.

- Growth Rate

각 feature map이 연결되어 있기 때문에 feature map의 채널 수가 많으면 채널을 각 채널별로 계속 연결할 수 있어 더 많은 채널로 이어질 수 있다. 그래서 DenseNet에서는 각 계층의 feature map에 매우 적은 수의 채널을 사용하는데, 이를 각 계층의 피쳐 맵에 대한 채널 수(k)라고 한다. 이러한 방식으로 다음 밑에서 Transition Layer을 소개하고자 한다.

- Bottleneck Layer

DenseNet은 ResNet과는 다른 Bottleneck Layer 구조를 가지고 있다.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/75468567-c0be7b80-59d0-11ea-93c9-ad5869a5bae8.png" width="75%"></p>

3 x 3 Convolution, 1 x 1 Convolution을 볼 수 있다. 여기서 입력 feature map의 채널 수만큼 만드는 것이 아니라, growth_rate만큼 feature map을 만들어 계산 비용을 줄일 수 있다. 이는 효율적으로 매개변수를 줄일 수 있다.

- Transition Layer

단순하게 말하면, feature map의 수를 줄이는 기능이다.

- Composite function

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/75469140-d54f4380-59d1-11ea-8317-273f6dcacf39.png" width="75%"></p>

다음은 pre-activation 구조로 BatchNormalization - ReLU function - Convolution Sequence를 사용함.

- avg pooling vs max pooling

이번 실험은 Max, Average Pooling을 비교할 것이다.

Pooling은 CNN에서 매우 중요한 역할을 한다. Subsampling을 사용하여 위치나 이동에 강한 특성을 가진 feature map의 크기를 줄일 수 있다.

Average Pooling 방법은 많은 ReLU를 활성 함수로 사용한다. 이로 인해 0이 많이 발생하며, 평균 작동에 의한 강한 자극이 감소한다. 또한 강한 자극과 약한 자극의 평균을 취하면 상쇄되는 상황이 발생할 수 있다.

반면에 Max Pooling 방법은 학습 데이터에 지나치게 적합할 수 있다.

Stochastic Pooling도 추가적으로 있는 것을 알아 두자.

DenseNet 121 Layer을 사용하였으며 Pooling에 대해서 비교하고자 하였다.

250epoch 학습하였을 경우, Avg Pooling은 Accuracy 86.87%, Loss 0.28962, Max Pooling은 Accuracy 90.85%, Loss 0.22649가 나왔다.

같은 에폭을 학습한 결과 Max Pooling은 학습 데이터에 더 빠르게 적합하는 것을 볼 수 있으며 Average Pooling 강한 부분과 약한 부분의 평균을 취하여 상쇄되는 상황이 발생하여 Max Pooling보다 못미치는 것을 볼 수 있었다.

#### 4. EfficientNet(Optimizer - SGD, Adam)
