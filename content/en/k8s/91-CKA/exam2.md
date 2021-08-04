
---
title: "CKA 시험 공부 2"
linkTitle: "exam2"
weight: 3
date: 2021-08-02
description: > 
  시험에 나오는 유형 정리 
---



<figure>
  <img src="/img/practice/cka/Certified Kubernetes Administrator for Kubernetes.jpg"
       alt="Certified Kubernetes Administrator for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>


## Node-1

별도 Cluster의 접근 없이 node-1에서 주어진 문제 풀이

---

### Etcd 스냅샷을 하는데 TLS를 이용

- Endpoint : https:127.0.0.1:2379
- Path : /snapshot/etcd.db
- Version : 3.2.12
- CA : /tls/ca.crt, Client Cert : /tls/client.crt, Client Key : /tls/client.key
 

`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#built-in-snapshot>

```sh
ETCDCTL_API=3 etcdctl --endpoints https://127.0.0.1:2379 --cacert /tls/ca.crt --cert /tls/client.crt --key /tls/client.key snapshot save /snapshot/etcd.db
ETCDCTL_API=3 etcdctl --write-out=table snapshot status /snapshot/etcd.db
```

내 Cluster에서 해볼려면..

```sh
cd /etc/kubernetes/pki/etcd 에 있는 ca.crt, server.crt, server.key 파일을 내용을 메모장에 복사한 후

아래 명령으로 Pod로 설치되어 있는 etcd로 들어갑니다.
kubectl exec -n kube-system etcd-k8s-master -it -- sh

그리고 해당 Pod 안에서 메모장에 복사해 놓은 ca.crt, server.crt, server.key파일을 만들고.

ETCDCTL_API=3 etcdctl --help 명령으로 --cacert, --cert, --key 들의 옵션 이름을 확인합니다.
ETCDCTL_API=3의 의미는 ectd version 3 api를 날린다는 의미인데 version마다 위에 옵션 이름이 달라짐.
예를들어 다른 버전에서는 --cacert를 --ca-file로 사용하기 때문에 --ca-file을 version 3 api 명령에 넣으면 에러가 남

그리고 아래 명령으로 snapshot을 만듬
ETCDCTL_API=3 etcdctl --endpoints https://127.0.0.1:2379 --cacert ca.crt --cert server.crt --key server.key snapshot save snapshot.db

마지막으로 확인
ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db
```


<br/>
<br/>

---
## ek8s Cluster
---

ek8s cluster에서 작업하는 문제
```sh
kubectl config use-context ek8s
```

<br/>
<br/>

### name=ek8s-node-0로 라벨이 달린 노드를 사용불가하게 만들고, 해당 노드위에 있는 Pod들은 모두 reschedule함
---


`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/>

```sh
kubectl get node --show-labels |grep name=ek8s-node-0
kubectl drain ek8s-node-0 --ignore-daemonsets --force
```

내 Cluster에서 해볼려면..

```sh
내 node1에 라벨 등록
kubectl label nodes k8s-node1 name=k8s-node1

등록된 라벨로 node 조회 
kubectl get node --show-labels |grep name=k8s-node1

Drain 하기 
kubectl drain k8s-node1 --ignore-daemonsets --force

확인
kubectl get node 

복구
kubectl uncordon  k8s-node1
```


<br/>
<br/>

---
## wk8s Cluster
---

wk8s cluster에서 작업하는 문제
```sh
kubectl config use-context wk8s
```

<br/>
<br/>

### wk8s-node-0의 State가 NotReady인데 원인을 찾아서 Ready상태로 되게 만들어라
---
- ssh wk8s-node-0
- sudo -i


`문제풀이`


```sh
SSH로 해당 Node 접근
ssh wk8s-node-0


해당 문제의 원인은 kubelet이 기동되지 않은 상태였음

kubelet 상태 확인
systemctl status kubelet

kubelet 기동
systemctl start kubelet
systemctl enable kubelet
```



<br/>
<br/>


### wk8s-node-1에 Static Pod를 만들어라
---
- ssh wk8s-node-1
- sudo -i
- image(nginx), name(static-web)
- file path : /etc/kubernetes/manifests

`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/tasks/configure-pod-container/static-pod/> 

```sh
해당 node에 접근해서 root권한으로 작업
ssh wk8s-node-1
sudo -i

문제에 주어진 파일 경로로 들어가서
cd /etc/kubernetes/manifests

yaml 파일을 만들어 놓음
cat <<EOF >static-web.yaml
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP
EOF
```

```sh
kubelet의 config.yaml 파일 찾기
find / -name kubelet
cd /var/lib/kubelet

해당 파일에 들어가서 staticPodPath 속성 및 경로 추가
vi config.yaml
---------------
staticPodPath: /etc/kubernetes/manifests
---------------

kubelet 재기동 
systemctl restart kubelet

그러면 kubelet이 재기동 되면서 /etc/kubernetes/manifests에 있는 yaml 파일들을 모두 실행함
```


<br/>
<br/>


---
## bk8s Cluster
---

bk8s cluster에서 작업하는 문제
```sh
kubectl config use-context bk8s
```

<br/>
<br/>

### bk8s의 Cluster 장애 문제
---

- ssh bk8s-master-0, bk8s-node-0
- sudo -i

`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/tasks/configure-pod-container/static-pod/>

```sh
문제의 원인은 Master Pod의 Static Pod들이 실행되지 않고 있었음.

해당 Master에 접근해서 root권한으로 작업
ssh bk8s-node-1
sudo -i

아래 경로에 파일들은 정상적으로 존재함
cd /etc/kubernetes/manifests

kubelet config 파일을 찾아서 내용을 확인해보면
cd /var/lib/kubelet
vi config.yaml

아래 설정에 path가 잘못되어 있기 때문에 제대로 수정함
staticPodPath: /etc/kubernetes/manifests

그리고 kubelet 재기동
systemctl restart kubelet

각 컴포넌트들이 정상적으로 기동되고 있는지 확인
kubectl get cs
```

<br/>
<br/>


---
## hk8s Cluster
---

hk8s cluster에서 작업하는 문제
```sh
kubectl config use-context hk8s
```

<br/>
<br/>

### hostPath타입의 PV를 만드는 문제
---

- name : task-pv-volume
- capacity : 10Gi
- accessModes : ReadWriteOnce
- volumeType : hostPath
- Path : /mnt/data

`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolume>

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume
  labels:
    type: local
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

```sh
위 내용으로 yaml파일을 만든다음 kubectl apply -f 파일명.yaml로 생성
```

<br/>
<br/>

---
## ik8s Cluster
---

22번쯤 나오는 문제이나 시간이 오래 걸리기 때문에 Skip하고 가장 마지막에 푸시기 바랍니다.

<br/>
<br/>

### ik8s-master-0과 ik8s-node-0을 이용해서 Cluster 설치
---
- kubeadm 이용
- Docker는 사전에 설치되어 있음
- --ignore-preflight-errors=all 옵션 추가
- 설치시 Config File 위치 : /etc/kubeadm.conf


`문제풀이`
<br/>
<https://v1-18.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/>
<br/>
<https://v1-18.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm>
<br/>
<https://v1-18.docs.kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/#init-workflow>


위에 링크들을 참고해서 설치


```sh
kubeadm init --ignore-preflight-errors=all --config /etc/kubeadm.conf
```

node join시  아래 명령 참고
master 설치 후 나오는 token URL에 --ignore-preflight-errors=all 부분 추가

```sh
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --ignore-preflight-errors=all --discovery-token-ca-cert-hash sha256:<hash>
```


