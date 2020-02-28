## MobileNet

### Abstract
모바일 및 임베디드 비전 애플리케이션을 위한 효율적인 모델.
경량 심층 신경망을 구축하기 위해 깊이 분리할 수 있는 심층분리를 사용하는 유선형 아키텍처 기반.
두 가지 간단한 글로벌 하이퍼 매개변수를 도입함으로써 적합한 크기의 모델을 선택 가능함.

#### 1. Introduction
더 높은 정확도를 달성하기 위해 더 깊고 복잡한 네트워크를 만드는 것이었다. 그렇지만 크기와 속도에 관하여 네트워크를 더 효율적으로 만드는 것은 아님.
따라서 모바일 및 임베디드 비전 애플리케이션의 설계 요건에 쉽게 부합할 수 있는 매우 작고 짧은 지연 시간 모델을 구축하기 위해 효율적인 네트워크 아키텍처와 두 개의 하이퍼파라미터 세트를 두 가지 섹션으로 설명함.
	- 소형 모델 제작에 대한 사전 작업을 검토.
	- 더 작고 더 효율적인 모바일 넷을 정의하기 위한 MobileNet 아키텍처와 두 개의 하이퍼 파라미터 폭 승수와 해상도 승수를 설명.

#### 2. Prior Work
효율적인 신경망을 구축하는 것에 미리 훈련된 네트워크를 압축하거나 소규모 네트워크를 직접 훈련시키는. 방법에 관심이 높아지면서 이 논문에서는 모델 개발자가 자신의 애플리케이션에 대한 리소스 제한(대기 시간, 크기)과 일치하는 작은 네트워크를 특별히 선택할 수 있도록 하는 네트워크 아키텍처 클래스를 제안함.
Mobilenets은 주로 지연 시간에 대한 최적화와 작은 네트워크에 초점을 맞추면서 속도에 대한 부분도 고려가 필요함.
이러한 소규모 네트워크를 구축하기 위해 여러 네트워크와 방법을 시연 하였으며 다른 접근법으로는 미리 훈련된 네트워크를 축소, 요인화(네트워크 속도 측면) 또는 압축(제품 정량화, 해싱, 가지치기, 벡터 정량화 및 허프만 코딩 등.)하는 것이었다.


#### 3. MobileNet Architecture
무방비 필터인 버블이라는 핵심 계층을 만들며 네트워크 구조에 대한 설명과 하이퍼파라미터(와이드, 해상도 멀티플레이어)를 분쇄하는 두 모델에 대한 설명을 하고자 함.

##### 3.1. Depthwise Separable Convolution


(a) standard convolution (b) depthwise convolution (c)1 x 1 pointwise convolution 계수하는 방법

standard convolution을 depthwise separable convolution으로 만드는 factorized convolutions의 한 형태인 depthwise convoulution, 1 x 1 convolution으로 불리는 pointwise convolution을 다루고자 함.
Convolution은 각 입력 채널에 단일 필터를 적용한 후 pointwise 결합은 1 x 1 convolution을 적용하여 출력의 깊이를 결정함.
깊이 있게 분리할 수 있는 경로는 이것을 다음 아래와 같이 두 개의 층으로 분리함.
	- 여과하기 위한 별도의 층
	- 결합하기 위한 별도의 층
이와 같은 구성들은 크기를 획기적으로 줄이는 효과가 있음.

	- standard convolution
다음 아래 식은 표준 컨볼루션을 위한 출력 피쳐 맵의 계산.



standard convolutional layer는 DF x DF x M 피쳐 맵 F를 입력하여 DF x N 피쳐 맵 G를 생성함.
standard convolutional 계층은 DK x DK x M x N 크기의 convolution kernel K에 의해 파라미터화되며 DK는 제곱으로 가정된 커널의 공간적 차원이고 M은 입력 채널의 수, N은 출력 채널의 수를 말함

계산 비용으로는 다음과 같음.



M, N, Dk x Dk 및 피쳐 맵 DF x DF에 따라 달라짐.
따라서 출력 채널의 수와 커널의 크기 사이의 상호작용을 끊기 위해 깊이 분리 가능한 convolution을 사용하며 커널을 기반으로 형상을 필터랑하고 새로운 표현을 만들기 위해 형상을 결합하는 효과를 가져옴.

	- depthwise separable convolution
깊이 분리할 수 있는 컨볼루션은 두 개의 레이어 depthwise,  pointwise convolution으로 구성함. 그래서 각 입력 채널(입력 깊이)당 하나의 필터를 적용하기 위해depthwise을 사용하며 pointwise convolution 1 x 1 convolution을 사용하여 깊이 계층의 출력의 선형 조합을 만듬.
배치 표준 및 ReLU 비선형 함수를 사용하며 입력 채널당 하나의 필터(입력 깊이)를 갖는 깊이 변환은 다음 아래 식과 같음.



K^은 크기 DK x DK x M의 깊이 있는 Convolutional kernel로 filter가 channel에 적용되어 여과된 출력 피쳐 맵의  channel에 들어감.

Depthwise convolution 계산 비용은 다음과 같음.



매우 효율적이지만 입력 채널만 필터링해 새로운 기능을 만들지는 않음.
그래서 새로운 형상을 만들어내기 위한 1 x 1 convolution을 통한 깊이 변환 출력의 선형 조합을 계산하는 추가 계층이 필요함.

Depthwise separable convolution 계산 비용은 다음과 같음.



이것은 깊이와 1 x 1 convolution의 합을 말함.

필터링과 결합의 두 단계 프로세스로 표현함으로써 계산의 감소를 얻을 수 있음.



3 x 3 depthwise separable convolution을 사용하여 standard convolution보다 8~9배 더 적은 연산을 사용함. 여기서 추가 factorized는 추가 연산을 크게 절약하지 못함.

##### 3.2. Network Structure and Training
구조로는 완전한 컨볼루션인 첫 번째 층을 제외하고는 depthwise separable convolution으로 구축함.
Depthwise separable, pointwise convolution 별도의 레이어로 간주하여 28개의 레이어를 가짐. 이것은 단순히 네트워크를 소수의 멀티 애드 단위로 정의하는 것만으로 충분하지 않고 일을 효율적으로 수행할 수 있도록 함.


왼쪽은 standard convolutional layer 오른쪽은 depthwise separable convolutions를 보여줌.



모델 구조에서 거의 모든 연산을 밀도 1 x 1 convolution에 있으며 이것은 고도로 최적화된 일반 매트릭스 곱셈(GEMM) 기능으로 구현될 수 있음.
GEMM은 수치 선형대수 알고리즘 중 하나로 매핑하기 위해 im2col이라고 불리는 메모리의 초기 재주문화가 필요함.

MobileNet 모델 Optimizers로 RMSprop 사용하며 훈련할 때 측면 헤드나 레이블 평활을 사용하지 않으며 작은 크기의 이미지를 제한하여 왜곡된 이미지를 추가로 감소시킨다.

##### 3.3. Width Multiplier: Thinner Models
모델을 더 작고 덜 계산적으로 비싼 만들기 위해 폭 승수라고 불리는  α를 도입함.
이 역할은 각 층에서 네트워크를 균일하게 가는  것으로 주어진 레이어 및 폭 곱셈  α의 경우 입력 채널 M의 수는 αM이 되고 출력 채널 N의 수는 αN이 됨.


MobileNet 전반적인 구성을 보여줌.

다음 아래 식은 폭 승수 α를 갖는 depthwise separable convolution 계산 비용을 확인할 수 있음.


이러한 폭 승수의 추가적인 구성으로 합리적인 정확도, 대기 시간 및 크기 트레이드 오프로 새로운 더 작은 모델을 정의할 수 있으며 처음부터 교육을 받아야 하는 새로운 축소된 구조를 정의하기 위해 사용함.

##### 3.4. Resolution Multiplier: Reduced Representation
신경망의 연산비를 줄이는 두 번째 하이퍼 파라미터는 분해능 곱셈기 ρ해상도 승수이다.
이것은 입력 이미지에 적용하고 모든 층의 내부 표현은 이후에 동일한 곱셈기에 의해 감소됨.


Convolution, depthwise separable conv, 폭 승수, 해상도 승수를 차례로 확인하면서 계산 비용에 대한 표현을 한 눈에 볼 수 있음.

#### 4. Experiments
계층 수보다는 네트워크의 폭을 줄임으로써 depthwise convolution 효과에 대한 부분을 조사함. 그런 다음 폭, 해상도 승수를 기반으로 네트워크 축소와 트레이드오프, 여러 가지 모델을 비교와 다른 응용 프로그램에 적용됨을 설명하고자 함.

##### 4.1. Model Choices


Standard convolution을 사용하였을 때와 depthwise separable convolution 사용할 때 정확도 부분에서 1%만 감소킨다는 것을 확인하면서 그의 비용적인 측면을 같이 보여줌.


더 얇은 모형 폭, 해상도 승수를 추가함으로써 정확도와 비용적인 측면으로의 효율을 같이 확인 할 수 있음. 자세한 설명은 생략 하겠음. - 표 참고.

##### 4.2. Model Shrinking Hyperparameters
아래 그래프는 위에 표를 시각화하여 보여주는 것으로 조금 더 편리하게 확인할 수 있도록 한번 더 보여줌.

16개 모델에 대한 Imagenet 정확도와 연산 간의 균형을 보여줌. 폭 승수 0.25에서 모델이 매우 작을 때 log linear with a jump 얻을 수 있었음.


16개 모델의 변수 수와 Imagenet 정확도 간의 트레이드오프를 보여줌.



바로 위에 보이는 여러 표들은 여러 모델과의 비교와 여러 기준을 적용하였을 경우에 대해서 말함.


###### 4.3, 4.4, 4.5, 4.6, 4.7 여러 가지 다른 응용 프로그램은 본 논문에서 확인하고자 함. 이하 Fine Grained Recognition, Large Scale Geolocalizaton, Face Attributes, Object Detection, Face Embeddings에 대한 부분은 생략하겠음.

#### 5. Conclusion
깊이 있는 분리가 가능한 컨볼루션에 기반한 MobileNets라는 새로운 모델 아키텍처를 제안으로 효율적인 모델로 이어지는 몇 가지 중요한 설계 결정을 조사함.
그리고 크기 및 지연 시간을 줄이기 위해 합리적인 양의 정확도를 상쇄함으로써 폭 승수 해상도 승수를 사용하여 더 작고 빠른 MobileNet을 구축하는 방법을 시연했음.
그 후 여러가지 모델 비교와 다양한 과제에 적용하였다.
정확도의 약간의 부분을 상쇄함으로써 비용적인 측면 Mult-Adds, Parameters에서 상당한 이익을 가져올 수 있었음을 보여준다.
