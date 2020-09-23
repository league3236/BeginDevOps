
# REF
- https://arisu1000.tistory.com/27859
- https://kubernetes.io/ko/docs/tasks/administer-cluster/coredns/


# 쿠버네티스 DNS

쿠버네티스에서는 클러스터 내부에서만 사용가능한 DNS를 설정해놓고 사용할 수 있다. 그래서 포드간 통신을 할때나 IP가 아닌 도메인을 설정해 두고 사용할 수 있다. 그렇게 한 클러스터에서 아용하던 yaml 파일에서 포드간 통신을 도메인으로 설정해 둔다면 별다른 수정없기 그대로 다른 클러스터로 가져가서 사용하는 것도 가능하다.

ip로 통신하도록 되어있다면 한곳에 세팅해놨던 yaml 파일을 다른곳으로 옮겨 가져가서 사용하려고 할때 그 클러스터에서 사용하는 ip 대역이 다른것이라면 그대로 사용할 수 가 없게 된다. 이럴때 설정이 도메인을 사용하도록 되어 있다면 별다른 수정없이 그대로 사용할 수 있다.

그 뿐만 아니라 일부의 경우에는 서비스디스커버리(service discovery) 용도로 사용할 수 있다. 전문적인 서비스디스커버리를 사용하려면 dns가 아니라 다른 솔루션들을 사용해야 하겠지만 간단한 경우라면 dns를 이용해서 할 수도 있다. 

특정 pod들에 접근하려할때 도메인을 통해서 접근하도록 설정되어 있다면 pod에 문제가 생겨서 재생성되거나 배포때문에 재생성될때 IP가 변경되더라도 자동으로 도메인에 변경된 pod의 IP가 등록되기 때문에 자연스레 새로 시작된 pod 쪽으로 연결하는 것이 가능하다

## 서비스 디스커버리를 위해 CoreDNS 사용하기

CoreDNS를 사용하기 위해서는 kubectl 버전이 1.9v 이상이여야 한다.

### CoreDNS 소개

CoreDNS는 쿠버네티스 클러스터의 DNS 역할을 수행할 수 있는, 유연하고 확장 가능한 DNS 서버이다. 쿠버네티스와 동일하게, CoreDNS 프로젝트도 CNCF가 관리한다.

사용자는 기존 디플로이먼트인 kube-dns를 교체하거나, 클러스터를 배포하고 업그레이드하는 kubeadm과 같은 툴을 사용하여 클러스터 안의 kube-dns대신 CoreDNS를 사용할 수 있다.

Service를 생성하면 대응하는 DNS entry가 생성된다.

### CoreDNS 설치

https://github.com/coredns/deployment/tree/master/kubernetes

- deploy.sh 및 core.yaml.sed

deploy.sh 는 kube-dns가 실행중인 클러스터에서 CoreDNS를 실행하기 위한 매니페스트를 생성하는 편리한 스크립트이다.

coredns.yaml.sed 파일을 템플릿으로 사용하여 ConfigMap 및 CoreDNS 배포를 생성 한 다음 CoreDNS 배포를 사용하도록 Kube-DNS 서비스 선택기를 업데이트 한다. 기존 서비스를 재사용하는 방식이므로 서비스 요청에 지장을 주지 않는다.

기본적으로 배포 스크립트는 기존 kube-dns 구성도 동등한 CoreDNS Corefile로 변환시킨다. -s 옵션을 제공하면 배포 스크립트가 kube-dns에서 CoreDNS로의 ConfigMap 변환을 진행한다.

해당 스크립트에서 kube-dns deployment나 replication을 자동으로 삭제해주지는 않는다. CoreDNS로 배포한뒤 수동으로 kube-dns를 삭제해 주어야 한다.

manifest를 먼저 깊게 검토하고 해당 클러스터에 맞는지 체크해야한다. 클러스터의 버전과 구축법에 따라 매니페스트 일부를 수정해야 할 수 있다.

kube-dns를 coredns로 교체하는데 필요한 명령은 아래와 같다.

```
$ ./deploy.sh | kubectl apply -f -
$ kubectl delete --namespace=kube-system deployment kube-dns
```

non_BRAC deployment에서는 아래의 내용을 따른다.

1. Remove the line serviceAccountName: coredns from 2. the Deployment section.
Remove the ServiceAccount, ClusterRole, and ClusterRoleBinding sections.

### kube-dns로 롤백하기

CoreDNS를 실행하는 Kubernetes 클러스터를 ㅏube-dns로 되돌리려는 경우 rollback.sh 스크립트를 진행한다.

```
$ ./rollback.sh | kubectl apply -f -
$ kubectl delete  --namepsace=kube-system deployment coredns
```

## 클러스터내에서 도메인사용해보기

### 형식

`{Service 명}.{Namespace 명}.svc.cluster.local`

### 예시

1. Pod조회

```
$ kubectl get pod
NAME                                      READY   STATUS    RESTARTS   AGE
pod/http-go-5c6f458dc9-wtpdq              1/1     Running   0          7m31s
```

2. Pod 내부 접속

```
$ kubectl exec -it http-go-5c6f458dc9-wtpdq -- bash
```

3. DNS를 활용한 Service 검색 1

```
root@http-go-5c6f458dc9-wtpdq:/usr/src/app# curl http-go-svc 
Welcome! http-go-5c6f458dc9-wtpdq
```

4. DNS를 활용한 Service 검색 2 - svc.cluster.local 생략가능

```
root@http-go-5c6f458dc9-wtpdq:/usr/src/app# curl http-go-svc.default
Welcome! http-go-5c6f458dc9-wtpdq
```

5. DNS를 활용한 Service 검색 3 - default Namespace 생략 가능

```
root@http-go-5c6f458dc9-wtpdq:/usr/src/app# curl http-go-svc
Welcome! http-go-5c6f458dc9-wtpdq
```

쿠버네티스에서 사용하는 내부 도메인은 service와 pod에 대해서 사용할 수 있고 일정한 패턴을 가지고 있다.

특정 서비스에 접근하는 도메인은 다음처럼 구성된다.

**aname**이라는 네임스페이스에 속한 **bservice**가 있다고 했을때 이 서비스에 접근하는 도멘인은 **bservice.aname.svc.cluster.local**이 된다. **bservice.aname** 순으로 서비스와 네임스페이스를 연결한 다음에 마지막에 **svc.cluster.local**을 붙이면 된다.

특정 pod에 접근하는 도메인은 다음처럼 구성된다.
default 네임스페이스에 속한 **cpod(10.10.10.10)**이라는 이름의 pod에 대한 도메인은 다음처럼 구성된다.

**10-10-10-10.default**

IP인 10.10.10.10에서 .을 -로 변경해서 사용하고 네임스페이스 이름인 default와 연결한 뒤에 pod.cluster.local을 붙여주면된다. 하지만 이렇게 하면 포드의 ip를 그대로 사용해야 하니깐 도메인 네임을 사용하는 장점이 사라지게 된다. 그래서 다른 방법을 사용할 수 있다. 포드를 실행할때 spec에 hostname와 subdomain을 지정해서 사용할 수 있다. 다음처럼 예제 yaml을 살펴보자. spec에 hostname와 subdomain을 지정한다.


vim testdns.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-simple-app
  labels:
    app: kubernetes-simple-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kubernetes-simple-app
  template:
    metadata:
      labels:
        app: kubernetes-simple-app
    spec:
      hostname: appname
      subdomain: default-subdomain
      containers:
      - name: kubernetes-simple-app
        image: arisu1000/simple-container-app:latest
        ports:
        - containerPort: 8080
```

이런 경우 이 포드에 접근할 수 있는 도메인은 appname-default-subdomain.default.svc.cluster.local으로 생성된다. hostname인 appname와 subdomain인 default-subdomain을 앞에 사용하고 네임스페인스인 default를 붙여준다음에 .svc.cluster.local을 붙여준다. 여기서 눈여겨봐야할점은 마지막에 붙인 .svc.cluster.local이 pod가 아니라 svc로 시작한다는 점이다.

해당 도메인을 사용하여 접근이 가능한지 살펴보자

```
$ kubectl apply -f testdns.yaml


$ kubectl get pods

NAME                                     READY   STATUS              RESTARTS   AGE
kubernetes-simple-app-55f6884cbb-rrw9z   0/1     ContainerCreating   0          3s
mymysql-8dddc55d6-z4pn9                  1/1     Running             0          5h13m
nginx-deployment-6bdf6857b5-dcr76        1/1     Running             0          35m

아래에서 에러가 발생한다..


$ kubectl exec kubernetes-simple-app-6695c7b497-wnz4g -- nslookup appname.default-subdomain.default.svc.cluster.local
nslookup: can't resolve '(null)': Name does not resolve

nslookup: can't resolve 'appname.default-subdomain.default.svc.cluster.local': Try again
```

원인 찾기

아래와 같이 정상적인 cluster pod안에 dns 설정부를 확인해보았다(katacoda playground)
```
master $ kubectl exec -it {PodName} -- nslookup kubernetes.default

example)

master $ kubectl exec -it kubernetes-simple-app-6695c7b497-q92cj -- nslookup kubernetes.default
nslookup: can't resolve '(null)': Name does not resolve

Name:      kubernetes.default
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
```

안되는 내 cluster에서는 위와 같이 응답이 나오지 않았고

pod의 ip를 찾아 직접적으로 nslookup을 해보았다

```
$ kubectl get pods -o wide
NAME                                     READY   STATUS    RESTARTS   AGE    IP             NODE               NOMINATED NODE   READINESS GATES
kubernetes-simple-app-6695c7b497-qcmrf   1/1     Running   0          178m   192.168.1.35   {nodehost}   <none>           <none>

$ kubectl exec -it kubernetes-simple-app-6695c7b497-qcmrf -- nslookup 192.168.1.35
nslookup: can't resolve '(null)': Name does not resolve

Name:      192.168.1.35
Address 1: 192.168.1.35 appname.default-subdomain.default.svc.k8s
```

appname.default-subdomain.default.svc.k8s

끝부분이 .default.svc.k8s 되어있다.

resolve.conf를 확인해보자

정상적인 cluster에서는 resolv.conf는 아래와 같이 되어있다

```
$ kubectl exec {pod name} cat /etc/resolv.conf
nameserver 10.96.0.10
nameserver 8.8.8.8
search default.svc.cluster.local svc.cluster.local cluster.local
```

그러나 내가 구성한 클러스터는 아래와 같이 설정되어있다.

```
$ kubectl exec -it kubernetes-simple-app-6695c7b497-qcmrf -- cat /etc/resolv.conf


nameserver 10.96.0.10
search default.svc.k8s svc.k8s k8s
options ndots:5
```

**search default.svc.k8s svc.k8s k8s**

이부분을 한번 변경해보자
다음과 같이 dns 설정을 바꾸는것이 가능하다

$ vim dnsutils.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: dnsutils
  namespace: default
spec:
  containers:
  - name: dnsutils
    image: gcr.io/kubernetes-e2e-test-images/dnsutils:1.3
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
  dnsPolicy: ClusterFirst
  dnsConfig:
    nameservers:
    - 8.8.8.8
    searches:
    - default.svc.cluster.local
    - svc.cluster.local
    - cluster.local
```

실행
```
$ kubectl create -f dnsutils.yaml
```

확인
```
$ kubectl exec -i -t dnsutils -- nslookup kubernetes.default
```

이래도 안되는구나..

확인해보니 
kubeadm init 부분이 잘못설정되어있었다

아래 내용을
```
$sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --service-cidr 10.96.0.0/12 --service-dns-domain "k8s" --apiserver-advertise-address {swarm-master-01 ip}
```

아래로 바꾸어서 reset후 k8s 구성을 다시 진행하였다...;;

```
$sudo kubeadm init --apiserver-advertise-address {master ip}
```

다시 띄우고
```
$ kubectl create -f dnsutils.yaml
```

다시 재확인
```
$ kubectl get pods

NAME                                     READY   STATUS    RESTARTS   AGE
kubernetes-simple-app-6695c7b497-hxp2d   1/1     Running   0          3m
```

정상 작동

```
$ kubectl exec -it kubernetes-simple-app-6695c7b497-hxp2d -- nslookup appname.default-subdomain.default.svc.cluster.local
```

nslookup을 통해서 mysql pod 접속가능한지 확인해보기

./mysql.md를 통해서 작업 후 실행

```
# kubectl get service -o wide
NAME                       TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE    SELECTOR
kubernetes                 ClusterIP   10.96.0.1      <none>        443/TCP          41m    <none>
mysql-cluster-ip-service   NodePort    10.105.17.80   <none>        3306:30006/TCP   3m5s   app=mysql
```

mysql-cluster-ip-service.default.svc.cluster.local 로 요청 보내보자

```
$ kubectl exec -it kubernetes-simple-app-6695c7b497-hxp2d -- nslookup mysql-cluster-ip-service.default.svc.cluster.local
nslookup: can't resolve '(null)': Name does not resolve

nslookup: can't resolve 'mysql-cluster-ip-service.default.svc.cluster.local': Try again
command terminated with exit code 1
```

확인을 해보니 서버의 방화벽 문제로 파악 

iptable로 NodePort의 방화벽 해지

## CoreDNS 기능

- 내부에서 DNS 서버 역할을 하는 Pod가 존재
- 각 미들웨어를 통해 로깅, 캐싱, Kubernetes 질의하는 등의 기능을 가짐

![dns1](./imgs/dns.png)

- 해당 DNS에는 `configmap` 저장소를 사용해 설정 파일을 컨트롤

```
$ kubectl get configmap coredns -n kube-system -o yaml

apiVersion: v1
data:
  Corefile: |   #ns 지정
    .:53 {
      errors
      health {
        lameduck 5s
      }
      ready
      kubernetes cluster.local inaddr.arpha ip6.arpa {
        pods insecure
        fallthrough in-addr.arpha ip6arpa
        ttl 30
      }
      prometheus :9153
      forward . /etc/resolv.conf {
        max_concurrent 1000
      }
      cache 30
      loop
      reload
      loadbalance
    }
    중략
```

Pod에서도 Subdomain을 사용하면 DNS 서비스 사용이 가능하다.

- 원래 CoreDNS는 Service까지만 조회 가능했으나, Pod에서도 Subdomain을 사용하면 DNS 서비스 사용 가능하다.
- yaml 파일의 hostname은 Pod의 `metadata.name`의 value를 따름 (필요한 경우 hostname을 따로 선택 가능)
- `subdomain` objet는 서브 도메인을 지정하는데 사용 -> 서브 도메인을 지정하면 FQDN 사용 가능
  - FQDN 형식 : {hostname}.{subdomain}.{namespace}.svc.cluster-domain.example
  - FQDN 예시 : busybox-1.default-subdomain.default.svc.cluster-domain.example