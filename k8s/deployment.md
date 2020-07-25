# 배포

## rolling update

가장 많이 사용되는 배포 방식 중 하나

새 버전을 배포하면서, 새 버전 인스턴스를 하나씩 늘려나가고, 기존 버전을 하나씩 줄여나가는 방식이다.

이 경우 기존 버전과 새 버전이 동시에 존재할 수 있는 단점이 있지만, 시스템을 무 장애로 업데이트할 수 있다는 장점이 있다.

롤링 업데이트를 하려면 RC를 두개 만들어야 하고, RC의 replica수를 단계적으로 조절해줘야 한다. 또한 배포가 잘못되었을때 순서를 뒤집어서 진행하여야 한다.

### Deployment

RC를 이용해서 구현하는 방법은 운영이 복잡해지는 단점이 있다.
그래서 쿠버네티스에서는 일반적으로 RC를 이용해서 배포하지 않고
Deployment라는 개념을 사용한다.
Replica를 사용하여 롤링업데이틀 구성하면 RC를 두개 만들어야 하고, 하나씩 Pod의 수를 수동으로 조정해야 하기 때문에 이를 자동화해서 추상화한 개념이 Deployment이다.


## ref
- https://bcho.tistory.com/1266