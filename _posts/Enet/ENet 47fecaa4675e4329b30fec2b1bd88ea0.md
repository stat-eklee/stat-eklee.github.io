---
title:  "[논문리뷰] ENet"
header:
  teaser: /assets/images/enet.png

categories:
  - Vision
tags:
  - Semgentation
last_modified_at: 2021-12-13T12:11:00-15:00
---

# ENet

+ 경량화 관련 지표 (속도)

1) Fps(**F**rames **P**er **S**econd) : 1초당 몇 프레임을 처리할 수 있는지를 말한다. 실시간(real-time) 사용 가능하려면 최소 30 fps 의 성능이 나와야 함. 

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled.png)

2) GFLOPs(**G**iga **FL**oating point **O**perations **P**er **S**econd) , FLOPs(**FL**oating point **O**perations **P**er **S**econd) : 초당 부동소수점 연산이란 의미로 1초동안 수행할 수 있는 부동소수점 연산의 횟수를 기준으로 삼음.

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%201.png)

참조 : [https://ko.wikipedia.org/wiki/플롭스](https://ko.wikipedia.org/wiki/%ED%94%8C%EB%A1%AD%EC%8A%A4)

3) Power consumption (참조 : ESPNet) 

4) Memory efficiency (참조 : ESPNet) 

# 0.  Abstract

- 모바일 애플리케이션에서 실시간으로 픽셀 단위 semantic segmentation을 수행하는 기능은 매우 중요
- 하지만, 최근의 CNN은 많은 부동 소수점 연산을 필요로 하고 실행 시간이 길어서 유용성을 저해하는 단점 존재.
- 짧은 지연 시간이 필요한 작업을 위해 특별히 만들어진 ENet(효율적인 신경 네트워크)이라는 새로운 아키텍처를 제안
- ENet은 최대 18배 빠르며 FLOP가 75배 적게 필요하고 매개 변수가 79배 적으며 기존 모델과 비슷하거나 더 나은 정확도를 제공
- CamVid, Cityscapes 및 SUN 데이터셋에서 테스트했으며 기존 방법과의 비교, 네트워크의 정확도와 처리 시간 간의 균형에 대해 보고
- 임베디드 시스템에서 제안된 아키텍처의 성능 측정값을 제시하고 ENet을 더욱 빠르게 만들 수 있는 가능한 소프트웨어 개선을 제안

# 1. introduction

- 증강현실 웨어러블(VR), 자율주행 차량에 대한 관심이 높아지면서 저전력 모바일 장치에서 실시간으로 작동할 수 있는 semantic segmentation(또는 visual scene-understanding) 알고리즘이 요구됨
- 대규모 데이터와 GPU의 발전으로 CNN이 고전적 컴퓨터 비전 알고리즘의 성능을 능가. CNN은 계속 발전하고 있지만 Segmentation에 적용하면 대략적인 공간 결과만을 제공하는 문제점 발생
- color based segmentation 또는 conditional random fields(CRF)와 같은 다른 알고리즘과 함께 사용하는 경우가 많음(예 : Deeplab v2)

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%202.png)

- 영상을 spatially하게 segmentation하기 위해, SegNet 또는 FCN과 같은 여러 개의 아키텍처가 제안(다중 클래스 분류를 위해 설계된 매우 큰 모델인 VGG16 아키텍처를 기반- 많은 매개변수와 긴 추론 시간 소요)
- 10fps 이상의 속도로 영상을 처리해야 하는 많은 모바일 또는 배터리 구동 애플리케이션에 사용 불가

![VGG16 architecture](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%203.png)

VGG16 architecture

- 빠른 추론과 높은 정확도에 최적화된 새로운 Neural Network 아키텍처(Enet)를 제안

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%204.png)

# 2. relate work

- Smantic segmentation은 영상의 내용을 이해하고 대상 개체를 찾는 것이 중요(자율주행, 증강현실 등의 응용 분야에서 가장 중요한 기술)
- 실시간 처리-운용이 필수적이기 때문에 CNN을 신중하게 설계하는 것이 중요함.
- State-of-the-art(SOTA) scene-parsing CNN은 encoder와 decoder라는 두 개의 별도 신경 네트워크 아키텍처를 함께 사용(Segnet)
- probabilistic auto-encoders에 영감을 받아, encoder-decoder 네트워크 아키텍처는 SegNet-basic에 도입되었으며, SegNet에서 더욱 개선
- 인코더는 입력을 분류하도록 훈련된 vanilla CNN(예: VGG16) decoder는 encoder출력을 업샘플링하는 데 사용
- 그러나, 이러한 네트워크는 큰 아키텍처와 수많은 매개 변수 때문에 추론속도가 느림
- FCN(Fully Convolution Network)과 달리, VGG16의 FC계층은 부동 소수점 작업과 메모리 공간을 줄이기 위해 최신 버전의 SegNet에서는 제거했음에도 불구하고 SegNet도 실시간 처리 불가
- 기존의 다른 아키텍처들은 더 간단한 classifiers를 사용하고 후처리 단계로 조건부 무작위 필드(CRF) 사용 > 부담스러운 후처리 단계를 사용하며 종종 프레임에서 더 적은 수의 픽셀을 차지하는 클래스(도로의 사람, 작은 사물)에 라벨을 붙이지 못하는 문제와 속도 저하 문제 발생

# 3. network architecture

- Table1의 구분선과 각 블록 이름 뒤의 숫자로 강조 표시된 것처럼 여러 단계로 나눠짐
- 512의 예제 입력 이미지 해상도에 대해 출력 크기(output size)

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%205.png)

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%206.png)

1) initial block : MaxPooling은 겹치지 않는 $2 \times2$ *windows size 에서 수행되며, conv($3 \times 3$*, stride2)에는 concatenation 후 최대 16개의 feature maps을 합한 13개의 필터가 있음.
2) bottleneck module(병목모듈) : conv는 $3 \times 3$개의 필터가 있는 정규, 확장 또는 완전 컨볼루션(디콘볼루션)이거나 두 개의 asymmetric으로 분해된 $5\times5$ conv.

그림 2b에서와 같이 단일 메인 분기 및 확장과 분리된 컨볼루션 필터를 가진 다음 요소별 추가 기능과 다시 병합하는 ResNets을 채택

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%207.png)

- Resnet12 구조
    
    ![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%208.png)
    

각 블록은 세 개의 convolutional 레이어로 구성

1. dimensionality를 감소시키는 $1 \times 1$의 projection
2. 주요 convolutional 레이어 (그림 2b의 convolutional 레이어)
3. $1 \times 1$의 expansion
- 배치 정규화(Regularizer)와 PReLU를 모든 컨볼루션 사이에 배치
- 원래 문서(ResNet) 와 마찬가지로 이를 병목 모듈이라고 함 : 병목 현상이 down-sampling되면 main branch에 최대 풀링(MaxPooling) 계층 추가
- 첫 번째 레이어 : $1 \times 1$  projection은 다음으로 대체 > $2 \times 2$  convolution with stride 2 in both dimensions. feature map의 수와 일치하도록 zero-pad 설정
- Conv 레이어 : $*3 \times 3*$ 필터를 사용하는 정규, 확장 또는 FC(deconvolution 또는 fractionally strided convolution)
- asymmetric convolution :  $5\times 1$과 *$1\times5$* 컨볼루션의 시퀀스로 대체할 때도 있음
- regularizer : 병목이 발생하기 전에는 p = 0.01, 이후에는 p = 0.1인 Spatial Dropout 사용.

+ PReLU (parametric ReLU) : 음수에 대한 gradien기울기(a)를 변수(parametric)로 두고 학습을 통하여 업데이트 시키는 방식

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%209.png)

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%2010.png)

초기 단계에는 단일 블록이 포함. 1단계는 5개의 병목 블록(bottleneck1.0이 1개, bottleneck1.x이 4개)으로 구성되며, 2단계와 3단계는 동일한 구조(2.0~2.8/ 3.0~3.8)
단, 3단계는 시작 부분에서 입력을 다운샘플링(downsampling)하지 않음(0번째 병목 현상 생략)

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%2011.png)

- cuDNN [29]은 convolution과 bias 추가를 위해 별도의 커널을 사용하기 때문에 커널 호출의 수와 전체적인 메모리 작업을 줄이기 위해 어떤 projections에서도 bias terms를 사용하지 않음 > 정확성에 아무런 영향을 미치지 않음
- 각 컨볼루션 계층과 비선형성 사이에서 배치 정규화를 사용
- 디코더에서 최대 풀링은 max unpooling으로 대체되고 패딩은 바이어스가 없는 spatial convolution으로 대체
- 최종 출력에 C feature maps (객체 클래스 수)이 있는 반면 초기 블록은 입력 프레임의 3개 채널에서 작동하기 때문에 마지막 업샘플링 모듈에서 pooling indices를 사용하지 않음
- 성능상의 이유로 네트워크의 마지막 모듈로 full convolution(fullconv)만 배치하기로 결정
- 이 모듈만으로도 decoder 처리 시간의 상당 부분을 차지.

# 4. Design choices

- ENet의 최종 아키텍처를 형성한 가장 중요한 실험 결과와 직관에 대해 설명

## 4.1 Feature map resolution

- semantic segmentation 중에 images 다운샘플링시 발생하는 두 가지 주요 단점

1) feature map resolution 감소 > exact edge shape과 같은 공간 정보의 손실을 의미

2) 전체 픽셀 segmentation에서는 출력의 resolution이 입력과 동일해야 함 > 강력한 다운샘플링이 강력한 업샘플링을 요구 > 모델 크기와 계산 비용을 증가시킴

- 해결 방안
    - FCN에서 인코더에 의해 생성된 feature map을 추가, SegNet에서 max-pooling계층에서 선택된 요소의 indices을 저장해 사용하여 디코더에서 sparse 업샘플링 맵을 생성하는 것으로 해결.
    - SegNet은 메모리 요구 사항을 줄일 수 있기 때문에 해당 접근 방식을 사용했지만 강력한 다운샘플링은 정확성을 해칠 수 있어 최대한 제한하려함.
    - 다운샘플링의 한 가지 큰 이점 : 다운샘플링된 이미지에서 작동하는 필터는 receptive field가 더 커서 컨텍스트를 더 많이 수집 가능
    - 예) 도로의 rider와 보행자와 같은 계층을 구분하려는 경우 특히 중요. 네트워크가 사람들의 외모와 그들이 보이는 맥락이 똑같이 중요하다는 것을 배우는 것만으로는 충분하지 않아서 이를 위해 dilated convolutions을 사용하는 것이 더 나을 수 있음.

## 4.2 Early downsampling

- 대용량 입력 프레임 처리 비용이 너무 크다는 점에 주목 : 많은 아키텍처들이 네트워크의 초기 단계의 최적화에 크게 관심을 기울이지 않고 있으나, 이는 종종 가장 시간이 많이 들 때도 있음.
- Enet의 처음 두 블록은 입력 크기를 크게 줄이고 feature map의 small set만 사용.
- 시각적 정보는 공간적으로 매우 중복되므로 보다 효율적인 표현으로 압축 가능.
- 초기 네트워크 계층이 분류에 직접적으로 기여해서는 안되나, 이는 오히려 좋은 feature 추출기 역할을 할 수 있음. 네트워크 후반에 16개에서 32개로 feature maps의 수를 늘린다고 해서 cityscape 데이터셋의 정확도가 향상되지는 않음.

## 4.3 Decoder size

- encoder-decoder 아키텍처에 대해 SegNet에서 제시된 것과는 다른 관점을 제공
- SegNet :  encode와 decoder가 매우 대칭적인 아키텍처로 구성.
- ENet 아키텍처는 대형 encoder와 소형 decoder로 구성 : encoder가 원래의 분류 아키텍처와 유사한 방식으로 작동할 수 있어야 한다는 아이디어에서 비롯됨 > 더 낮은 해상도의 데이터로도 작동하고 정보 처리 및 필터링을 제공하기 위해서
- 대신 decoder의 역할은 세부 정보만 미세 조정하면서 encoder의 출력을 업샘플링하는 것입니다.

## 4.4 Nonlinear operations

- 최근 논문[31]에서는 Convolution 전에 ReLU 및 배치 정규화 계층을 사용하는 것이 유익하다고 언급. > 이 아이디어를 ENet에 적용하려고 했지만 정확도에 좋지 않은 영향.
- 대신 네트워크의 초기 계층에서 대부분의 ReLU를 제거하면 결과 개선.
- 원인확인 : 네트워크의 모든 ReLU를 PReLUs로 교체. 이는 비선형성의 음의 기울기를 학습하는 것을 목표로 feature map당 추가 매개변수를 사용. identity가 preferable한 transfer function인 계층에서, PReLU 가중치는 1에 가까운 값을 가지며, 반대로 ReLU가 선호되는 경우에는 0에 가까운 값을 가질 것으로 예상
- 파란색 선은 가중치 평균이며, 최대 weight와 최소 weight사이의 영역은 회색으로 표시됩니다. 각 수직 점선은 main branch의 PReLU에 해당하며 각 병목 블록 사이의 경계를 표시.
- 67번째 모듈의 회색 수직선은 인코더와 디코더 경계임.

![네트워크 깊이 대비 PReLU weight 분포.](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%2012.png)

네트워크 깊이 대비 PReLU weight 분포.

- 초기 계층 가중치는 큰 편차를 나타내며 양의 값으로 약간 편향되어 있는 반면 인코더의 후반부에서는 반복 패턴으로 안정화됨.
- branch의 모든 계층은 일반 ReLU와 거의 똑같이 동작하는 반면, 병목 모듈 내부의 가중치는 음수. 즉, 음수 값을 반전시키고 scales down 하는 기능이므로 깊이가 제한된 Enet의 architecture에서 identity가 잘 작동하지 않았다는 가설을 세움.
- 이렇게 loss function이 학습되는 이유는 원래 ResNets가 수백 개의 계층 깊이를 가질 수 있는 네트워크인 반면, ENet네트워크는 몇 개의 계층만 사용하며 정보를 신속하게 필터링해야 하기 때문일 수 있음.
- 디코더 가중치가 훨씬 더 positive(양수)가 되고 identity 에 더 가까운 기능을 학습한다는 것은 주목할 만함.
- 이를 통해 업샘플링된 출력을 fine-tuning하는데만 디코더가 사용된다는 직감을 확인함.

## 4.5 Informatation-preserving demensionality changes

- 입력을 조기에 다운샘플링해야 하지만, 급격한 dimensionality 감소로 인해 정보 흐름이 저해될 수 있음.
- 이 문제에 대한 매우 좋은 접근방식은 [28] “Rethinking the inception architecture for computer vision,”  논문에 제시되어 있음.
- VGG 아키텍처에서 사용되는 방법, 즉 풀링 수행 후 dimensionality 확장에 따른 컨볼루션이 상대적으로 저렴하더라도 표현 병목 현상을 유발한다는 주장이 제기됨. (또는 필터를 더 많이 사용하도록 하여 컴퓨터 효율성을 떨어뜨림.)
- 반면, 특성 맵 깊이를 증가시키는 컨볼루션 이후의 풀링은 계산 비용이 많이 소요.
[28]에서 제시한 stride 2의 convolution과 병렬로 풀링을 수행하고 결과 feature map을 연결하기로 선택
- 이 기술을 통해 초기 블록의 추론 시간을 10배 높일 수 있었음.
- 또한 원래 ResNet 아키텍처에서 한 가지 문제가 발견. 다운샘플링시, 컨볼루션 분기의 처음 11개 투영은 두 dimensionality 모두에서 2의 stride로 수행되며, 이는 입력의 75%를 효과적으로 폐기함.
- 필터 크기를 $2 \times 2$로 늘리면 전체 입력을 고려할 수 있으므로 정보 흐름과 정확도가 향상.
물론 이러한 4배 많은 계층은 컴퓨팅 비용이 더 많이 들지만 Enet에는 이러한 계층이 너무 적기 때문에 오버헤드가 눈에 띄지 않음.

## 4.6 Factorizing filters

- 비대칭 컨볼루션  : 컨볼루션 가중치는 상당한 양의 중복성을 가지며, 각 $n \times n$ **conv*은 두 개의* 작은 conv **으로 분해 가능
- 하나는 $1 \times n$필터가 있고, 다른 하나는 $1 \times n$ 필터가 있음
- 네트워크에서 $n = 5$인 비대칭 컨볼루션을 사용했으므로, 이 두 작업의 비용은 단일 $3 \times 3$ 컨볼루션과 유사
- 블록에서 학습한 다양한 functions을 늘리고 receptive field 증가
- 병목 모듈에 사용되는 일련의 작업 (projection, convolution,projection) 하나의 큰 컨볼루션 레이어를 일련의 작고 단순한 작업으로 분해하는 것으로 볼 수 있음. 즉, low-rank 근사.
- 이러한 인수분해는 큰 속도 상승을 가능하게 하며, 매개 변수의 수를 크게 줄여 중복을 줄임[32].

## 4.7 Dilated Convolutions

- 네트워크는 wide receptive Field 를 가지는 것이 매우 중요하므로, 보다 넓은 맥락을 고려하여 분류를 수행할 수 있음
- Feature map의 과도한 다운샘플링을 피하며 모델을 개선하기 위해 dilated convolutions을 사용
- 가장 작은 resolutions으로 작동하는 단계에서 여러 병목 모듈 내부의 주요 convolutional 레이어를 교체
- cityscapes의 IoU를 약 4% 포인트 높여줌
- 순차적으로 배치하는 대신 다른 병목 모듈(정규 및 비대칭)과 interleaved할 때 최고 정확도 얻음

## 4.8 Regularization

- 대부분의 pixel-wise segmentation dataset은 상대적으로 작으므로($10^3$개 images) 신경 네트워크와 같은 표현력이 뛰어난 모델이 빠르게 과적합되기 시작.
- 초기 실험에서는 L2 weight decay를 사용했지만 성공하지 못함. 이후 stochastic depth를 시도했고, 이는 정확도를 높임.
- 그러나 전체 분기를 삭제하는 dropout은 임의의 subset을 선택하는 대신 모든 채널 또는 채널 중 어느 것도 무시되지 않는 Spatial Dropout [27]을 적용하는 특별한 경우라는 것이 명백해짐
- Spatial Dropout 추가 직전, 전체 분기를 삭제하는 것을 컨볼루션 분기 끝에 배치했을 때, stochastic depth 보다 훨씬 더 잘 작동하는 것으로 나타남.

# 5. results

- 3개의 서로 다른 데이터셋에서 ENet의 성능을 벤치마킹하여 실제 애플리케이션에 대해 정확도와 실시간성을 입증
- 도로 장면의 CamVid, Cityscapes 데이터셋, 실내 장면의 SUN RGB-D
- SegNet[11]은 가장 빠른 분할 모델 중 하나이며, FCN보다 훨씬 적은 파라미터와 더 적은 메모리만 사용하므로 이 모델을 기준으로 함.
- ENet 모델, training, 테스트 및 성능 평가 스크립트는 CUDNN 백엔드가 있는 Torch7 기계 학습 라이브러리 사용
- 결과 비교를 위해 클래스 평균 정확도(mean Acc)와 IoU(Intersection-over-union) 메트릭스 사용

## 5.1 Performance Analysis

- 사용 : NVIDIA Titan X GPU /  NVIDIA TX1 임베디드 시스템 모듈
- 입력 이미지 크기 640 × 360으로 NVIDIA TX1 보드에서 10fps 이상을 달성하도록 설계
- Inference 를 위해 모든 네트워크의 속도를 높이기 위해 배치 정규화와 드롭아웃 계층을 컨볼루션 필터에 **병합**

### Inference time

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%2013.png)

- 표 2는 resolution 이 다양한 단일 입력 프레임에 대한 추론 시간(inference time)을 비교. 초당 처리 가능한 프레임 수도 추가됨.
- -는 메모리가 부족하여 측정을 수행할 수 없음을 표시.
- ENet은 SegNet보다 훨씬 빠르며 실시간 애플리케이션에 높은 frame rates을 제공하고 인코더-디코더 아키텍처와 함께 매우 심층적인 신경망 모델을 실질적으로 사용 가능.

### Hardware requirements

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%2014.png)

- 표 3은 서로 다른 모델에서 사용되는 floting point 연산 및 parameter 수를 비교한 것.
- ENet의 requirements가 두 자릿수보다 작기 때문에 ENet의 효율성은 명백.
- model size(fp16) :모델 파라미터를 half precision floating point format(fp16)으로 저장하는 데 필요한 저장공간.
- ENet에는 parameter 가 너무 적어서 필요한 공간이 0.7MB에 불과. 이것은 임베디드 안의 매우 빠른 온칩 메모리에서 전체 네트워크 구동 가능.

### Software limitations

- 이러한 수준의 성능에 도달할 수 있도록 한 가장 중요한 기술 중 하나는 **convolutional layer factorization**.
- 그러나, 이 방법을 적용하면 floating point operations 수와 parameters 수를 크게 줄일 수 있었지만, 개별 커널 호출 수를 증가시켜 각각을 더 작게 만듬.
- 작업 중 일부가 cheap해져 GPU kernel launch의 비용이 실제 계산 비용을 초과하기 시작할 수 있다는 것을 발견.
- 또한 kernel 은 이전 값에 의해 레지스터에 보관된 값에 접근하지 못하기 때문에 실행 시 global memory에서 모든 데이터를 로드하고 작업이 끝나면 저장해야 함. 즉,즉, 기능 맵을 지속적으로 저장하고 다시 로드해야 하므로 커널 수가 많아지면 메모리 트랜잭션 수 증가.
- 이는 비선형 연산의 경우에 특히 명확해짐. ENet에서 PReLU는 inference time 의 1/4 이상을 소비. 하지만 PReLu는 단순한 point-wise 연산일 뿐이고 병렬화가 매우 쉽기 때문에, 저자는 앞에서 언급한 데이터 수집에 의해 발생한다고 가정.
- 이는 심각한 limitations이지만 기존 소프트웨어에서 kernel fusion을 수행함으로써 해결 가능. 즉, 즉, 컨볼루션의 결과에 비선형성을 직접 적용하거나 한 번의 호출로 다수의 작은 컨볼루션을 수행하는 커널을 생성하는 것. cuDNN과 같은 GPU 라이브러리의 이러한 개선은 네트워크의 속도와 효율성을 더욱 높일 수 있음.

## 5.2 Benchmarks

- 네트워크 훈련을 위해 Adam 최적화 알고리즘[35]을 사용. (이를 통해 ENet은 매우 빠르게 수렴 가능)
- 4개의 Titan X GPU를 사용하여 training을 받은 모든 데이터셋에 대해 불과 3~6시간밖에 걸리지 않음.
- 먼저 입력 이미지의 다운샘플링 영역을 분류하도록 인코더만 훈련한 다음 디코더를 추가하고 업샘플링 및 픽셀 단위 분류를 수행하도록 네트워크를 training.
- 5e-4의 learning rate와 2e-4의 L2 weight decay, 10의 batch size 가 지속적으로 최상의 결과를 제공
- $w_{class} = \frac{1}{ln(c + p_{class})}$로 정의된 custom class weighing scheme 방식을 사용. inverse class probability weighing와 대조적으로, 가중치는 0에 접근하는 확률에 따라 제한됨. c는 1.02로 설정한 additional hyper-parameter이다(즉, class weights를 [1, 50]의 간격으로 제한한다).

### cityscapes

- 5000개의 labeled 이미지로 구성. 이 중 2975개는 training 에, 500개는 validation에, 나머지 1525개는 test 세트로 구분[14].
- 뛰어난 품질과 매우 다양한 도로 시나리오, 종종 많은 보행자와 자전거 타는 사람들이 등장하기 때문에 cityscapes는 가장 중요한 벤치마크 중 하나.
- ofﬁcial evaluation scripts에서 선택된 19개 클래스에 대해 training [14].
- instance-level intersection over union metric(iIoU)라는 추가 메트릭을 사용, 이는 평균 객체 크기로 가중치를 주는 IoU.
- 표 4에서와 같이 ENet은 IoU 및 iOU 클래스는 물론 IoU에서 SegNet을 능가. ENet은 현재 Cityscapes 벤치마크에서 가장 빠른 모델. validation 세트의 예측 예는 그림 4.

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%2015.png)

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%2016.png)

### Camvid

- 367개의 training 이미지와 233개의 test이미지가 포함[15].
- 건물, 나무, 하늘, 자동차, 도로 등 11개의 클래스가 있는데, 12번째 클래스에는 라벨이 부착되지 않은 데이터가 포함, training시 이를 무시.
- 이 데이터의 원래 프레임 해상도는 960×720이지만 training 전에 이미지를 480×360으로 다운샘플링함. 표 5에서 우리는 ENet의 성능을 기존의 SOTA와 비교.

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%2017.png)

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%2018.png)

### Sun RGB-D

- 실내 객체 클래스가 37개인 5285개의 training 이미지와 5050개의 test 이미지로 구성.
- 본 연구에서는 깊이 정보를 활용하지 않고 RGB 데이터에 대해서만 네트워크를 training.
- 표 6에서는 ENet의 성능을 SegNet [11]과 비교.
- global average accuracy와 IoU에서는 낮지만 class average accuracy에서는 비슷.
- iIoU 메트릭의 도입을 주목 [14]. class average accuracy의 비교 결과는 Enet이 SegNet만큼 작은 개체를 구별할 수 있음을 나타냄.
- ENet은 실시간으로 이미지를 처리할 수 있으며 임베디드 플랫폼의 SegNet보다 거의 20배 더 빠르기 때문에 이 차이는 작다고 연구자는 주장.

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%2019.png)

![Untitled](ENet%2047fecaa4675e4329b30fec2b1bd88ea0/Untitled%2020.png)

# 6. Conclusion

- semantic segmentation을 위해 처음부터 설계된 새로운 신경망 아키텍처를 제안.
- 주요 목표는 fully fledged deep learning workstations에 비해 부족한 자원을 embedded platforms에서 효율적으로 사용하는 것.
- 연구는 계산 및 메모리 요구 사항이 훨씬 큰 기존 기준 모델과 비슷한 성능이거나 초과하는 성능을 보이면서 큰 차이를 제공.
- NVIDIA TX1 하드웨어에 대한 ENet의 적용은 real-time portable embedded solutions의 예시.