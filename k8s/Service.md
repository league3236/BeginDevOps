# Service

쿠버네티스 파드는 수명이 있다. 파드는 생성되고, 소멸된 후 부활하지 않는다. 만약 앱을 실행하기 위해 디플로이먼트를 사용한다면, 동적으로 파드를 생성하고 제거할 수 있다.

각 파드는 고유한 IP 주소를 갖지만, 디플로이먼트에서는 한 시점에 실행되는 파드 집합이 잠시 후 실행되는 해당 파드 집합과 다를 수 있다.

이는 다음과 같은 문제를 야기한다. (“백엔드”라 불리는) 일부 파드 집합이 클러스터의 (“프론트엔드”라 불리는) 다른 파드에 기능을 제공하는 경우, 프론트엔드가 워크로드의 백엔드를 사용하기 위해 프론트엔드가 어떻게 연결할 IP 주소를 찾아서 추적할 수 있다.

Pod 집합에서 실행중인 애플리케이션을 네트워크 서비스로 노출하는 추상화 방법, 쿠버네티스를 사용하면 익숙하지 않은 서비스 디스커버리 메커니즘을 사용하기 위해 애플리케이션을 수정할 필요가 없다. 
쿠버네티스는 Pod에게 고유한 IP 주소와 Pod 집합에 대한 단일 DNS 명을 부여하고, 그것들 간의 load-balance를 수행할 수 있다.

## service resource

쿠버네티스에서 서비스는 파드의 논리적 집합과 그것들에 접근할 수 있는 정책을 정의하는 추상적 개념이다. 서비스가 대상으로 하는 파드 집합은 일반적으로 셀렉터가 결정한다.

예를 들어, 3개의 replica로 실행되는 스테이트리스 이미지 - 처리 백엔드를 생각해보자. 이러한 레플리카는 대체 가능하다. 즉, 프론트엔드는 그것이 사용하는 백엔드를 신경쓰지 않는다. 백엔드 세트를 구성하는 실제 파드는 변경될 수 있지만, 프론트엔드 클라이언트는 이를 인식할 필요가 없으며, 백엔드 세트 자체를 추적해야할 필요도 없다.

## Cloud Native Service Discovery

애플리케이션에서 서비스 디스커버리를 위해 쿠버네티스 API를 사용할 수 있는 경우, 서비스 내 파드 세트가 변경될 때마다 업데이트 되는 엔드포인트를 API서버에 질의할 수 있다.

## 서비스 정의

쿠버네티스의 서비스는 파드와 비슷한 REST 오브젝트이다. 모든 REST 오브젝트와 마찬가지로, 서비스 정의를 API 서버에 POST하여 새 인스턴스를 생성할 수 있다. 서비스 오브젝트의 이름은 유효한 DNS 서브도메인 이름이어야 한다. 

예를 들어, 각각 TCP 포트에서 9376에서 수신하고 `app=MyApp` 레이블을 가지고 있는 파드 세트가 있다고 가정해보자

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

해당 명세는 "my-service"라는 새로운 서비스 오브젝트를 생성하고, `app=MyApp` 레이블을 가진 Pod의 TCP 9376 포트를  
