## kaggle_plant_pathology2020
- Google에서 제공해주는 Colab gpu 환경에서 다음과 같이 실험을 진행하였음.

- 이 대회는 Computer Vision and Pattern Recognition Conference(CVPR 2020)에서 Fine-Grained Visual Categorization FGVC7 워크숍 일부로써 kaggle Web Site를 통하여 데이터를 제공 받음.

### 문제
농작물에 영향을 미치는 많은 질병의 오진은 화학 물질의 오용으로 이어질 수 있으며, 내성 병원균 균주의 출현, 투입 비용 증가, 심각한 경제적 손실 및 환경 영향으로 더 많은 발생이 이어질 수 있다. 따라서 컴퓨터 비전 기반 모델로 감염된 조직의 나이, 유전적 변이 및 나무 내의 빛 상태로 인한 증상의 큰 변화 등을 알고자 함.

### 목표 
사과 잎의 사진이 주어지면 건강 상태를 정확하게 평가할 수 있을까?

=> 이 대회는 건강에 좋은 잎, 사과 녹에 감염된 잎, 사과 녹병이 있는 잎 및 둘 이상의 질병을 가진 잎을 구별하는데 의미를 둠.

    1. 많은 질병을 정확하게 구별하고, 한 잎에서 하나 이상의 질병을 정확하게 구별.
    2. 희귀한 질병과 새로운 증상을 다룸.
    3. 깊이 인식 - 각도, 빛, 그늘, 잎의 생리적 연령에 대응.
    4. 학습 중 관련 기능 검색하기 위한 식별, 주석, 수량화 및 컴퓨터 비전 안내에 전문적인 지식을 통합.


### 데이터 정보

<p align="center">Plant Pathology Class</p>
<p align="center"><img src="https://user-images.githubusercontent.com/45933225/82079740-4fb07980-971e-11ea-9c9e-c5e74ac4709e.png" width="100%"></p>

    데이터 포맷 : jpg, jpeg
    총 3642장, Train : 1821장 Test : 1821장(Test data label 없음).
    해상도 : 2048 * 1365 * 3
    class : healthy(건강), multiple_diseases(복합 질병), rust(녹병), scab(붉은곰팡이병)
    
    
- Train data graph

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/81252299-8fad9780-9060-11ea-8c17-99b1abd089ef.png" width="75%"></p>

=> combinations 516개, healthy 91개, rust 622개, scab 592개 총 Train data 1821개 확인.

#### 데이터 분류

    1. Train data 각 클래스 별로 8:2 비율로 나누어서 validation data 생성.   -  모델의 성능을 일반화하기 위해서 validation data을 이용(이상적으로  8:2 비율이 적절하다고 함)
    2. 교차 검증(Cross Validation) KFold 적용.    -   모델이 데이터를 판단하기 위한 데이터 수가 적은 경우 검증 성능의 신뢰도가 떨어지는 것을 해결하고자 함(반대로 정확도 향상을 위한것으로도 말할 수 있음)
    
#### 데이터 보완
데이터의 수가 부족하여 데이터를 보완하느 방법으로 이미지의 레이블을 변경하지 않고 픽셀 변화시키는 CNN의 과적합 방지와 성능 향상을 위하여 Data Augmentation의 albumentations Moudle을 사용하였음.

다음과 같이 데이터에 적용함.

    Train data : Resize, RandomCrop, Resize, Flip, ShiftScaleRotate, HorizontalFlip
    Validation data : Resize
    Test data : Resize
    
추가로 정확도 향상과 과적합 방지를 하고자 교차 검증(KFold)를 적용하여 데이터 보완을 시도하였음.   
    
### 실험 소개 및 수행 내역
문제 해결 방법으로 Computer Vision(CV)의 분야에서 사용하는 Convolution Neural Network(CNN)을 이용하여 컴퓨터가 영상(이미지 및 비디오) 디지털 이미지의 특징을 학습하여 특정 패턴을 사용한 이미지를 분류하고자 함.

다음과 아래의 순서대로 실험을 수행함.

    - 1. 데이터에 적합한 모델 찾기(VggNet, ResNet, DenseNet, EfficientNet).
    - 2. 데이터에 적합한 레이어층 찾기 (Model(v1, v2 etc..)
    - 3. 기존 데이터 분류와 데이터 보완 및 과적합 방지를 위한 교차 검증 적용(K-Fold)의 비교
    - 4. Filter 가중치 부여 Attention의 기법 적용(Default(None), Squeeze and Excitation, Convolutional Block Attention Module)
    - 5. Optimizers 선정(SGD, RMSProp, Adam)
    - 6. 하이퍼파라미터 최적화(Batch Size, Image Size etc..)

### 1. 데이터에 적합한 모델 
###### ./select_model 실행 코드 확인 가능
대표적으로 Convolution Neural Network에서 사용하는 모델을 선택하여 실험하였으며 모델을 제외한 나머지 조건들을 동일하게 설정하여 비교하고자 하였음.

모델은 다음과 같이 사용함.

    - VggNet, ResNet, DenseNet, EfficientNet

하이어파라미터 초기화

      - image size(height, width, channel) : 342, 512, 3
      - epoch : 100, 150
      - batch size(train, validation, test) : 8, 8, 1
      - step(train, validation, test) : 80, 20, 1
      - label : one-hot encoding    -   using Categorical entropy
      - optimizer : Adam(learning rate : 0.001)  -   validation loss에 맞게 laerning rate 조정하였음.
      - Augment
        Train data : Resize, RandomCrop, Resize, Flip, ShiftScaleRotate, HorizontalFlip
        Validation data : Resize
        Test data : Resize
      - Dropout : x
        
위에서 제시하는 조건을 동일하게 하여 각 아래 모델들을 학습하였음.

##### Confusion Matrix
Train + Validation data를 이용하여 임시 테스트 함.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/83384955-43693380-a423-11ea-88b5-dd9cc0df0086.png" width="60%"></p>

각 모델마다 전체적으로 데이터 수가 적은 multiple_diseases 클래스의 예측률이 현저히 떨어지는 것을 확인할 수 있었음. 그 외 클래스들은 예측률에 있어서 큰 차이가 없었음.

##### Train, Validation 데이터의 각 Accuracy, loss를 비교한 그래프.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/82093860-e38e3f80-9736-11ea-99a8-41147d15090b.png" width="60%"></p>

<table border="1">
	<a>&nbsp;각 모델의 결과 다음 아래와 같이 확인할 수 있음(Train, Validation data 전체데이터를 이용하여 학습한 임시 Test 결과).&nbsp;</a>
	<th>Model</th>
	<th>Vlidation Max Accuracy</th>
	<th>Validation Min Loss</th>
	<th>Test Accuracy</th>
	<th>Test Loss</th>
	<th>Kaggle Accuracy(100, 150 epoch)</th>
	<tr>
	    <td>VggNet(Layer 16)</td>
	    <td>0.9750</td>
	    <td>0.1019</td>
   	    <td>0.9750</td>
  	    <td>0.1130</td>
	    <td>0.9360, 0.9370</td>
	</tr>
	<tr>
            <td>ResNet(Layer 50)</td>
	    <td>0.7562</td>
	    <td>0.5031</td>
   	    <td>0.67326</td>
  	    <td>0.61080</td>
	    <td>X, 0.5630</td>
	</tr>    
    <tr>
	    <td>DenseNet(Layer 121)</td>
	    <td> X </td>
	    <td> X </td>
	    <td> X </td>
  	    <td> X </td>
	    <td> X </td>
	</tr>
    <tr>
	    <td>EfficientNet(Layer b0)</td>
	    <td>0.9750</td>
	    <td>0.1019</td>
	    <td>0.94393</td>
  	    <td>0.1600</td>
	    <td>0.9480, 0.9460</td>
	</tr>
    </table>
    
일단 전반적인 히스토리를 보면 기존 논문에서 제시하는 모델의 성능만큼 효율적인 학습 결과를 얻을 수 없었음. 그래서 논문에서 제시하는 모델들이 모든 데이터에 맞는 것은 아니라는 것을 알았음.

결과 : VggNet, EfficientNet 두 모델이 데이터셋에 최적의 가중치에 빠르고 정확하게 수렴하여서 데이터에 맞는 모델 후보로 생각했으며 EfficientNet의 Resolution, Width, Depth의 3가지면에서 다양함을 시도할 수 있다고 생각하여 모델을 결정하게 되었음.


### 2. 데이터에 적합한 레이어층 찾기
###### ./find_layer 실행 코드 확인 가능.
EfficientNet paper에서 제공하는 b0 ~ b7의 순서대로 학습  비교하여 데이터셋에 적합한 레이어층을 찾고자 하였음.

하이퍼파라미터 초기화

      - image size(height, width, channel) : 342, 512, 3
      - epoch : 100
      - step(train, validation, test) : 80, 20, 1
      - label : one-hot encoding    -   using Categorical entropy
      - optimizer : Adam(learning rate : 0.001)  -   validation loss에 맞게 laerning rate 조정하였음.
      - Augment
        Train data : Resize, RandomCrop, Resize, Flip, ShiftScaleRotate, HorizontalFlip
        Validation data : Resize
        Test data : Resize
        
colab환경에 만족하여 batch size는 각 레이어층에 맞게 최대값을 주었으며 나머지는 위에서 제시하는 조건을 동일하게 설정하여 학습하였음.

#### EfficientNet b0 ~ b7 비교.

##### efficientnet b0 ~ b3
    batch size(train, validation, test) - b0 : (30, 30, 1), b1 : (24, 24, 1), b2 : (20, 20, 1), b3 : (16, 16, 1)

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/83259303-c35c8700-a1f2-11ea-9194-01080e099d3a.png" width="100%"></p>

<table border="1">
	<a>&nbsp;각 레이어의 결과는 다음 아래와 같이 확인할 수 있음(Train, Validation data 전체데이터를 이용하여 학습한 임시 Test 결과).&nbsp;</a>
	<th>Layer</th>
	<th>Vlidation Max Accuracy</th>
	<th>Validation Min Loss</th>
	<th>Test Accuracy</th>
	<th>Test Loss</th>
	<th>Kaggle Accuracy(100 epoch)</th>
	<tr>
	    <td> b0 </td>
	    <td>0.9616</td>
	    <td>0.1048</td>
   	    <td>0.9752</td>
  	    <td>0.0688</td>
	    <td>0.9565</td>
	</tr>
	<tr>
            <td> b1 </td>
	    <td>0.9625</td>
	    <td>0.1044</td>
   	    <td>0.9621</td>
  	    <td>0.1010</td>
	    <td>0.9100</td>
	</tr>    
    <tr>
	    <td> b2 </td>
	    <td>0.9550</td>
	    <td>0.1225</td>
	    <td>0.9654</td>
  	    <td>0.1038</td>
	    <td>0.9490</td>
	</tr>
    <tr>
	    <td> b3 </td>
	    <td>0.9718</td>
	    <td>0.0868</td>
	    <td>0.9670</td>
  	    <td>0.0853</td>
	    <td>0.9420</td>
	</tr>
    </table>

##### efficientnet_b4 ~ b7

50epoch 넘게 학습을 했지만 모델 레이어층이 깊어짐에 따라 최적화 값의 방향을 찾지 못한다고 생각하여 중간에 학습을 중단함. 

결과 : 점점 레이어층이 깊어짐에 따라 Vanishing gradient problem가 발생과 원본 이미지의 해상도 크기 감소로 고유의 픽셀 정보가 끝까지 전달을 못하여 모델이 특징 인식을 하지 못하는 것으로 예상함. 그래서 기존 환경을 고려하여 배치 사이즈와 이미지 사이즈에서 폭 넓게 조정할 수 있고 결과적으로 제일 높게 나온 b0 레이어층을 선택하기로 함.

### 3. 기존 데이터 분류와 데이터 보완 및 과적합 방지를 위한 교차 검증 적용(K-Fold)의 비교
###### ./Data Classification 실행 코드 가능.
꼭 정해진 기준은 아니지만 일반적으로 많이 알려져 있는 학습 데이터를 학습, 검증 데이터로 분류하는 적절한 비율로 8 : 2를 나눴으며 기존 데이터 분류에 해당함. 그리고 정확도 향상과 데이터 과적합 방지를 위하여 데이터 수를 보완하고자 많이 사용되고 있는 교차 검증의 KFold 방식을 테스트 하고자 함.

하이퍼파라미터 초기화

      - image size(height, width, channel) : 342, 512, 3
      - epoch : 100
      - step(train, validation, test) : 80, 20, 1
      - label : one-hot encoding    -   using Categorical entropy
      - optimizer : Adam(learning rate : 0.001)  -   validation loss에 맞게 laerning rate 조정하였음.
      - Augment
        Train data : Resize, RandomCrop, Resize, Flip, ShiftScaleRotate, HorizontalFlip
        Validation data : Resize
        Test data : Resize
	
위 레이어층 하이퍼 파라미터 동일하게 이어감.
	
	batch size(train, validation, test) - Baisc Data : (30, 30, 1), Cross Validation Data : (30, 5, 1)
	
##### Confusion Matrix

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/83427175-48ea6c00-a46b-11ea-8698-db4cfa2fd1e7.png" width="80%"></p>

##### Train, Validation 데이터의 각 Accuracy, loss를 비교한 그래프.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/83424534-7d5c2900-a467-11ea-9832-88d495c6443a.png" width="60%"></p>

<table border="1">
	<a>&nbsp;데이터 분류 결과는 다음 아래와 같이 확인할 수 있음(Train, Validation data 전체데이터를 이용하여 학습한 임시 Test 결과).&nbsp;</a>
	<th>Data Classification</th>
	<th>Vlidation Max Accuracy</th>
	<th>Validation Min Loss</th>
	<th>Test Accuracy</th>
	<th>Test Loss</th>
	<th>Kaggle Accuracy(100 epoch)</th>
	<tr>
	    <td> Basic </td>
	    <td>0.9616</td>
	    <td>0.1048</td>
   	    <td>0.9752</td>
  	    <td>0.0688</td>
	    <td>0.9565</td>
	</tr>
	<tr>
            <td> Cross Validation </td>
	    <td>1.0000</td>
	    <td>0.0231</td>
   	    <td>0.9604</td>
  	    <td>0.1109</td>
	    <td>0.9392</td>
	</tr>    
    </table>


검증 데이터를 학습 데이터에 추가하여 사용함으로써 Validation Accuracy, Loss의 고유의 목적을 잃어버림.
    
결과 : 기존 학습에 제공하는 전체 데이터 수가 적어서 학습에 미비하다고 생각이 되며 결과적으로 원하는 성능(정확도 향상과 과적합 방지)에 대한 부분을 얻지 못했으며 일반적인 학습법에 비해 시간 소요가 크다라는 단점을 가지고 있어 기존 데이터 방식을 유지해서 진행하고자 함.  

### 4. Filter 가중치 부여 Attention의 기법 적용

##### Confusion Matrix

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/83465169-0302c780-a4ae-11ea-8782-00c69d603153.png" width="80%"></p>

##### Train, Validation 데이터의 각 Accuracy, loss를 비교한 그래프.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/83465191-19108800-a4ae-11ea-9c23-7bbbe43e8775.png" width="100%"></p>


<table border="1">
	<a>&nbsp;각 레이어의 결과는 다음 아래와 같이 확인할 수 있음(Train, Validation data 전체데이터를 이용하여 학습한 임시 Test 결과).&nbsp;</a>
	<th>Attention</th>
	<th>Train Max Accuracy</th>
	<th>Train Min Loss</th>
	<th>Vlidation Max Accuracy</th>
	<th>Validation Min Loss</th>
	<th>Test Accuracy</th>
	<th>Test Loss</th>
	<th>Kaggle Accuracy(100 epoch)</th>
	<tr>
	    <td>Default(None)</td>
	    <td>0.0.9779</td>
	    <td>0.0649</td>
	    <td>0.9616</td>
	    <td>0.1048</td>
   	    <td>0.9752</td>
  	    <td>0.0688</td>
	    <td>0.9565</td>
	</tr>
	<tr>
            <td>Squeeze-and-Excitation Block</td>
	    <td>0.9962</td>
	    <td>0.0177</td>
	    <td>0.9850</td>
	    <td>0.0520</td>
   	    <td>0.9906</td>
  	    <td>0.0288</td>
	    <td>0.9539</td>
	</tr>    
    <tr>
	    <td>Convolutional Block Attention Module</td>
	    <td>0.9962</td>
	    <td>0.0177</td>
	    <td>0.9750</td>
	    <td>0.0653</td>
	    <td>0.9895</td>
  	    <td>0.0296</td>
	    <td>0.9628</td>
	</tr>
    <tr>
	    <td> b3 </td>
	    <td>0.9718</td>
	    <td>0.0868</td>
	    <td>0.9670</td>
  	    <td>0.0853</td>
	    <td>0.9420</td>
	</tr>
    </table>
