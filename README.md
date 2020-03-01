# CNN(Convolutional Neural Network)
본 CNN 정리는 라온피플(Laon People)님 블로그 https://blog.naver.com/laonple/220587920012 의 사이트를 참고하여 문제가 될 경우 삭제 하겠습니다.

신경망을 통해 학습을 하게 되면 어려운 많은 문제들을 해결이 가능함.

영상에 기반한 인식 알고리즘에서 좋은 결과를 얻으려면, 사전에 많은 처리 과정이 필요하기 때문에 multi-layered neural network를 바로 적용하는 것은 어려움이 따름.

### Multi-layered Neural Network의 문제점(특히, 비전에 적용에 따른..)
training data만을 넣어주는게 아닌, prior knowledge을 이용해 네트워크의 구조를 특수한 형태로 변형을 시켜주는 것.

그렇다면 2-D 이미지가 갖는 특성을 최대한 이끌어낼 수 있는 방법은?
(2-D의 기본적인 내용은 생략함.)

기존 신경망은 위 그림 처럼, 픽셀 값의 약간의 이동만 있더라도 새로운 training data로 처리를 해줘야 하는 문제점을 가지고 있었음.
또한 입력해주는 데이터가 항상 같은 경우가 아니기 때문에 좋은 결과를 얻기는 더욱 어려웠음.

결과적으로 multi-layered neural network의 문제점은 이미지의 topology(컴퓨터 네트워크의 요소들(링크, 노드 등)을 물리적으로 연결해 놓은 것.)을 고려하지 않고 row data에 많은 training data를 필요로 하고, 학습 시간을 필요로 함.
ex) 32 x 32 image => 2^32*32 무수한 양의 패턴이 나옴. - 그래서 전처리 과정이 따로 필요.

결과적으로 기존의 fully-connected muylti-layered neural network을 사용하면 다음과 같은 3가지 측면에 문제가 발생함.

    - 학습 시간(Training time)
    - 망의 크기(Network size)
    - 변수의 개수(Number of free parameters)
    
##### 위와 같은 문제를 해결하고자 CNN (Convolutional Neural Network) 이다.

CNN 들어가기 전에, Receptive Field(수용 영역)에 대해서 알아야 함.
사전 의미로는 간단하게 말하자면 다음과 같음.

    - 정보처리와 관계되는 포에 대한 응답을 일으키는 자극의 영역.
    
쉽게 말하자면 외부 자극이 전체 영향을 끼치는 것이 아니라 특정 영역에만 영향을 준다는 뜻임.
그래서 이게 '인식 알고리즘'으로 이어짐.

    - 영상 전체 영역에 대해 서로 동일한 연관성(중요도)로 처리하는 대신에 특정 범위에 한정 처리를 한다면 훨씬 좋은 결과를 얻을 수 있다라는 짐작.
    
Convolution 이란?
'신호 및 시스템'과정에서의 convolution은 특정 시스템에 입력이 가해졌을 때 시스템의 반응이 어떻게 되는지 해석하기 위한 용도로 사용.
'영상 처리 분야'에서 convolution은 주로 filter 연산에 사용이 되며, 영상으로부터 특정 feature들을 추출하기 위한 필터를 구현할 경우 사용.  - 3 x 3 window or mask or kernel을 영상 전체에 대해 반복적으로 수행을 하게 되며 그 계수(weight)값들의 따라 적정한 결과를 얻을 수 있음.

자세한 내용은 생략 하겠음. - 영상 처리 부분, 신호 및 시스템 분야를 공부하자ㅎㅎ

#### CNN의 특징

Locality(Local Connectivity) - 수용 영역과 유사하게 local 정보를 활용. 공간적으로 인접한 신호들에 대한 correlation 관계를 비선형 필터를 적용하여 추출해냄.

Shared Weights - 동일한 계수를 갖는 filter를 전체 영상에 반복적으로 적용함으로 변수의 수를 획기적으로 줄일 수 있으며, 항상성을 얻을 수 있음.

#### CNN의 구조 및 과정

<img width="735" alt="스크린샷 2020-03-01 23 45 45" src="https://user-images.githubusercontent.com/45933225/75627787-c60d0780-5c16-11ea-8db6-329e879b8ca0.png">

1. 특징을 추출
2. topology 변화에 영향을 받지 않도록 함.
3. 분류기 단계


