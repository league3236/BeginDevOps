
# ref 
- https://medium.com/@Alibaba_Cloud/gpu-sharing-scheduler-extender-now-supports-fine-grained-kubernetes-clusters-b610b2c030b2

대부분 kubernetes 서비스(클라우드 공급업체 포함) 모두 NvidiaGPU 컨테이너를 예약하는 기능을 제공한다. 일반적으로는 컨테이너에 GPU Card를 할당하게 된다. 이는 격리를 가능하게 하고 GPU를 사용하는 애플리케이션이 다른 애플리케이션의 영향을 받지 않도록 한다. 딥 러닝 모델 학습 시나리오에는 적합하지만 모델 개발 및 모델 예측 시나리오에는 낭비가 된다. 수요는 더 많은 예측 서비스가 동일한 GPU 카드를 공유하도록 허용하여 클러스터에서 NvidiaGPU 사용률을 향상시키는 것이다. 이를 위해서는 GPU resource 분할 작업이 필요한다. 일반적으로 클러스터 수준 GPU 공유는 다음 두 가지에 관한것이다.

1. 스케줄링
2. 격리

주로 스케줄링에 대해서 설명하며, 격리 솔루션은 향후 Nvidia MPS를 기반으로 구현된다.

## 사용자 시나리오

- 클러스터 관리자로서 클러스터의 GPU 활용도를 높이고 싶다. 그리고 개발 중에 여러 사용자가 모델 개발 환경을 공유한다.

- 애플리케이션 개발자로서 Volta GPU에서 동시에 여러 논리 작업을 실행할 수 있다.

