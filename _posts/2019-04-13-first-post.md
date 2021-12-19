---
title:  "[논문리뷰] ENet"
excerpt: "Semantic Segmentation"

categories:
  - Vision
tags:
  - segmentation
last_modified_at: 2021-12-13T12:11:00-15:00
---


ENet

+ 경량화 관련 지표 (속도)

1) Fps(**F**rames **P**er **S**econd) : 1초당 몇 프레임을 처리할 수 있는지를 말한다. 실시간(real-time) 사용 가능하려면 최소 30 fps 의 성능이 나와야 함. 

2) GFLOPs(**G**iga **FL**oating point **O**perations **P**er **S**econd) , FLOPs(**FL**oating point **O**perations **P**er **S**econd) : 초당 부동소수점 연산이란 의미로 1초동안 수행할 수 있는 부동소수점 연산의 횟수를 기준으로 삼음.

3) Power consumption (참조 : ESPNet) 

4) Memory efficiency (참조 : ESPNet) 

# 0.  Abstract

- 모바일 애플리케이션에서 실시간으로 픽셀 단위 semantic segmentation을 수행하는 기능은 매우 중요
- 하지만, 최근의 CNN은 많은 부동 소수점 연산을 필요로 하고 실행 시간이 길어서 유용성을 저해하는 단점 존재.
- 짧은 지연 시간이 필요한 작업을 위해 특별히 만들어진 ENet(효율적인 신경 네트워크)이라는 새로운 아키텍처를 제안
- ENet은 최대 18배 빠르며 FLOP가 75배 적게 필요하고 매개 변수가 79배 적으며 기존 모델과 비슷하거나 더 나은 정확도를 제공
- CamVid, Cityscapes 및 SUN 데이터셋에서 테스트했으며 기존 방법과의 비교, 네트워크의 정확도와 처리 시간 간의 균형에 대해 보고
- 임베디드 시스템에서 제안된 아키텍처의 성능 측정값을 제시하고 ENet을 더욱 빠르게 만들 수 있는 가능한 소프트웨어 개선을 제안
