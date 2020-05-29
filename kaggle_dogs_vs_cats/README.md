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
###### 자세한 내용은 각 항목의 제목을 클릭하면 결과를 확인이 가능함.

단순한 모델 학습이 아닌 각 모델마다 비교하고자 하는 것을 하나 정하여서 하고자 함.

그래서 다음과 아래와 같이 순서대로 비교하였음. 

    - 1. VggNet(Accuracy - Featrue Map)
    - 2. ResNet(Layer - 50, 101 Layer)
    - 3. DenseNet(Pooling - Max, Avg)
    - 4. EfficientNet(Optimizer - SGD, Adam)

#### 1. VggNet(Accuracy - Featrue Map)

- Accuracy

VggNet 16Layer의 기본 구성을 위 그림에서 확인할 수 있다. 실험에서 차이점으로는 각 블록에 Kernel_initializeer = 'he_normal'을 추가하였으며Vanishing/Exploding을 완하하기 위해 Convolution과 Max Polling 사이에 Batch Normalization을 추가 하였다. 그리고 출력층에 대해서 Multi Class가 아니여서 softmax가 아닌 binary class에 적합한 sigmoid함수를 사용하였다.

모델 컴파일의 경우 최적화 방법은 SGD(learning rate = 0.001, momentum = 0.9)함수를 사용했으며 손실 함수는 binary_crossentropy로 설정하였다. 그리고 하이퍼 파라미터는 (train, validation)batch_size 20, image_size 224 x 224, epoch 250, (train, validation)step 80, 20으로 설정하였다.

실험 결과로는 Accuracy 95.05%, Loss 0.1351이 나오는 것을 확인 하였다.

- feature map

 accuracy 95.05%, loss 13.51% feature map

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74591625-d345c600-505c-11ea-8f0f-56a6d300223e.png" width="75%"></p>

 accuracy 49.45%, loss 69.32% feature map

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74591647-038d6480-505d-11ea-96e9-5193de6d5b6f.png" width="75%"></p>

높은 정확도는 이미지의 특징을 찾아서 점점 특정 부분에 찾아가며 뚜렷하지만 낮은 정확도를 기록한 것을 보면 전체적으로 스무딩한 이미지를 볼 수 있으며 가끔 어떤 층에서는 특징을 못찾은 것도 볼 수 있었다.

#### 2. ResNet(Layer - 50, 101 Layer)

- 50, 101 Layer

Hyper parameter 초기화는 (train, validation)batch_size 20, epoch 550, Image size 224 x 224 추가적으로 Data Augmentation을 줬으며. 
(train, validation)step은 80, 20으로 설정하였다.

학습 결과 101 Layer 경우 Accuracy 92.70%, Loss 0.1951가 나왔으며, 50 Layer Accuracy 93.80%, Loss 0.1692가 나오는 것을 볼 수 있었으며 확연한 차이는 없지만 이 데이터에서는 50 Layer가 더 적합하다고 볼 수 있었으며 레이어층이 무조건 깊다고 좋은 학습 결과를 받을 수 있는 것은 아니었다.

- feature map(Conv_2D Layer)
다음은 입력층 부터 출력층까지 Convolution Layer층의 feature map을 보여준다.

50-layer

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/75012532-82532900-54c5-11ea-93c4-2ca3511520c0.png" width="75%"></p>

101-layer

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74638370-1ca62a80-51af-11ea-9a4c-d18b593d081a.png" width="75%"></p>
<p align="center"><img src="https://user-images.githubusercontent.com/45933225/74638389-2465cf00-51af-11ea-8884-f7839e414b94.png" width="75%"></p>

비교적 짧은 50 Layer에서도 충분히 이미지에 특징을 찾아가는 것을 확인이 가능하다.

#### 3. DenseNet(Pooling - Max, Avg)

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

optimizer(SGD, Adam)구조에서 다음과 같이 진행하였다.  - learning rate : 0.001 로 계속 돌림(비교가 목적이었음).

Hyper parameter 초기화는 (train, validation)batch_size 20, epoch 300, Image size 224 x 224 추가적으로 Data Augmentation을 줬으며. 
(train, validation)step은 80, 20으로 설정하였다.

위와 같이 optimizer를 제외한 나머지 설정을 똑같이 하였을 때, SGD의 결과는 accuracy 94.65%, loss 0.1321가 나왔으며 Adam은 accuracy 95.90%, loss 0.1053이 나왔다. 근소하지만 그래도 Adam이 최적값에 더 빠르게 수렴하면서 더 높은 성능을 얻을 수 있었다.

Grad CAM을 확인하면 특징 추출하는 부분에서는 확연한 차이가 있었다.

SGD

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/81701360-db1ed600-94a4-11ea-9915-0ed6bc760365.png" width="50%"></p>

Adam

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/81701018-66e43280-94a4-11ea-9b51-aeee87bb9dfb.png" width="50%"></p>






SGD옵티마이저를 보면 아예 
