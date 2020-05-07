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

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/81299146-bdbac800-90b0-11ea-8919-33dc07db0cbf.png" width="75%"></p>

위 그림에서 전체 이미지가 부분적으로 예측됨. 자세히 보면 영상의 노락색 영역은 파란색 영역을 사용하여 예측함.
즉 이미지 경계에서 이미지는 미러링에 의해 얻어짐.

출력 분할 맵의 매끄러운 타일을 얻기 위해서는 입력 타일 크기를 선택하는 것이 중요함.
즉, 모든 2 x 2 Max-pooling 작업이 균등한 x와 y 사이즈의 레이어에 적용되도록 해야 함.

##### Elastic Deformation for Data Augmentation

<p align="center"><img src="https://user-images.githubusercontent.com/45933225/81300529-bf858b00-90b2-11ea-97be-84a24d78afbd.png" width="75%"></p>

훈련 세트가 작아서 세트의 크기를 증가시키기 위해, 입력 및 출력 이미지 세그먼테이션 맵을 임의로 변형함으로써 수행함.

##### Separation of Touching Objects

