## U-Net(Biomedical Image Segmentation)
Biomedical Image Segmentation에서 유명한 FCN(Fully Convolutional Networks)중 하나로 각 이미지에 주석을 달기 위해 관련 지식을 가지고 있는 전문가 도움이 필요하다. 이런 시간의 소비를 줄이고자 주석 프로세스가 자동으로 이루어지면 사람의 노력을 줄이고 비용을 절감 할 수 있다. 또는 사람의 실수를 줄이기 위해 보조 역할을 할 수 있다.

다음은 전자 현미경(EM)이미지를 segment/annotate(분할/주석)화 한다.

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/81296649-2dc74f00-90ad-11ea-9168-766f65a2edaa.png" width="50%"></p>

#### U-Net Network Architecture

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/81297096-c3fb7500-90ad-11ea-9e2e-dce8873dbc07.png" width="75%"></p>

위 그림에서 U-Net Architecture 보여줌.
Contraction path(수축 경로), Expansion path(확장 경로)를 확인할 수 있음.

##### Contraction path(수축 경로) - Down sampling
3 x 3 Conv 2회 및 2 x 2 Max-pooling 연속 수행됨. - 고급 기능을 추출하는데 도움이 되지만 피쳐 맵의 크기는 줄어듬.

##### Expansion path(확장 경로) - Up sampling
2 x 2 Up-Conv 3 x 3 Conv 2회 연속 수행하여 피쳐 맵의 크기를 복구함. - "what'을 증가시키지만 "where"를 감소시킴.
즉, 고급 기능을 얻을 수 있지만, 현지화 정보는 잃어 버림.

따라서 Up samling 이후에는 동일한 수준의 피쳐 맵을 제공하기 위해 수축 경로에서 확장 경로로 현지화 정보를 제공함.

결과적으로 출력 피쳐 맵은 2개의 클래스, 셀 및 막만을 갖기 때문에 1 x 1 Conv 전환으로 피쳐 맵의 크기를 64에서 2로 맵핑함.

##### Overlap Tile Strategy

