
---
title: "CKA 시험 공부 1"
linkTitle: "exam1"
weight: 2
date: 2021-08-02
description: > 
  시험에 나오는 유형 정리1
---



<figure>
  <img src="/img/practice/cka/Certified Kubernetes Administrator for Kubernetes.jpg"
       alt="Certified Kubernetes Administrator for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>



## 사전 작업 
---

node-1에 kubectl 명령 자동완성 설정

https://v1-18.docs.kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-autocomplete

```sh
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

```sh
alias k=kubectl
complete -F __start_kubectl k
```

<br/>
<br/>


## k8s Cluster - 17문항
---

k8s cluster에서 작업하는 문제 (아래 명령어는 문제에서도 알려줌)
```sh
kubectl config use-context k8s
```

<br/>
<br/>

### Pod에서 특정 Error Log 문자열만 출력해서 파일 저장  
---

`사전환경`

```sh
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: poo
spec:
  containers:
  - name: container
    image: kubetm/init
    command: ['sh', '-c', 'echo Hello Kubernetes ; echo error-message; echo end; sleep 1000']
EOF
```


`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/reference/kubectl/cheatsheet/>

```sh
kubectl logs poo | grep error-message > ./poo 
```
<br/>
<br/>

### 모든 PV list를 이름순으로 정렬
---
`사전환경`

```sh
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv1
spec:
  capacity:
    storage: 1G
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /hostPath
    type: DirectoryOrCreate
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv2
spec:
  capacity:
    storage: 1G
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /hostPath
    type: DirectoryOrCreate
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv3
spec:
  capacity:
    storage: 1G
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /hostPath
    type: DirectoryOrCreate
EOF
```


`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/reference/kubectl/cheatsheet/>

```sh
kubectl get pv --all-namespaces --sort-by=.metadata.name > ./pvname
```

<br/>
<br/>


### 멀티 컨테이너 Pod만들기 (nginx, busybox, redis 등 제시됨)
---


`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pod-templates>


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: nginx
    image: nginx
  - name: busybox
    image: busybox
  - name: redis
    image: redis
```

```sh
kubectl apply -f multi.yaml
kubectl describe pods myapp-pod
```


<br/>
<br/>



### NodeSelector를 이용해서 Pod를 특정 Node에 생성

사전에 특정 node에 Label이 달려있음 key=value를 알려줌

---

`사전환경`

```yaml
kubectl label nodes k8s-node1 disktype=ssd
kubectl get nodes -l disktype=ssd
```


`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/concepts/configuration/assign-pod-node/#step-two-add-a-nodeselector-field-to-your-pod-configuration>


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disktype: ssd
```

```sh
kubectl apply -f node-selector.yaml
kubectl get pods nginx -o wide
```



<br/>
<br/>




### 사전환경 파일에 init 컨테이너 내용을 추가, 내용은 /workdir/init.txt 파일을 만들기
해당 Pod의 전반적인 작동내용은 init 컨테이너에서  /workdir/init.txt가 잘 만들어졌으면 Pod가 만들어지고 안만들어졌으면 종료

- initContainer란 : 본 Container가 실행되기 전에 사전 작업이 필요할 경우 initContainer를 사용하며, initContainer가 성공적으로 작업이 완료 된 후에 본 Container가 실행됨 

---

`사전환경`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  volumes:
  - name: workdir
    emptyDir: {}
  containers:
  - name: nginx
    image: nginx
    command: ["sh", "-c", "if [ -f /workdir/init.txt ] ; then sleep 1000; else exit 1; fi"]
    volumeMounts:
    - name: workdir
      mountPath: /workdir
```


`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/tasks/configure-pod-container/configure-pod-initialization/>


```yaml
  initContainers:
  - name: install
    image: busybox
    command:
    - "touch /workdir/init.txt"
    volumeMounts:
    - name: workdir
      mountPath: "/workdir"
```

<br/>
<br/>


### 제시된 pod에 Service 연결하기

---

`사전환경`

```sh
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: exist-pod
  labels:
    exist: pod
spec:
  containers:
  - name: nginx
    image: nginx
EOF
```


`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/tasks/access-application-cluster/service-access-application-cluster/>


```sh
kubectl expose pod exist-pod --name=cluster-service --type=ClusterIP --port=8080
```


<br/>
<br/>


### 제시된 Namespace위에 Pod 만들기 
- namespace : ns-01

---

`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/concepts/workloads/pods/pod-overview/#pod-templates>


```sh
kubectl create ns ns-01
```

``` yaml
apiVersion: v1
kind: Pod
metadata:
  namespace: ns-01
  name: ns-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
```

```sh
kubectl apply -f 9.yaml
kubectl get pods -n ns-01
```


<br/>
<br/>



### 특정 Namespace에 속해있는 서비스에 연결된 Pod List이름 저장하기
- Namespace : namespace-01
- Service name : service-nm

---

`사전환경`

```sh
kubectl create ns namespace-01
cat <<EOF | kubectl create -n namespace-01 -f -
apiVersion: v1
kind: Pod
metadata:
  name: pod-ns-1
  labels:
     ns: pod
spec:
  containers:
  - name: container
    image: kubetm/init
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-ns-2
  labels:
     ns: pod
spec:
  containers:
  - name: container
    image: kubetm/init
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-ns-3
  labels:
     ns: pod
spec:
  containers:
  - name: container
    image: kubetm/init
---
apiVersion: v1
kind: Service
metadata:
  name: service-nm
spec:
  selector:
    ns: pod
  ports:
  - port: 9000
EOF
```


`문제풀이`


```sh
kubectl get svc -n namespace-01 service-nm -o wide
kubectl get pods -l ns=pod -n namespace-01
vi pod-namelist.txt
```


<br/>
<br/>


### 조건에 따라 Pod을 만들되 Volume은 Not Persistent 하도록 함
-  Name (test-pd), Image(k8s.gcr.io/test-webserver), Namespace(volume-1)
-  Mount Path : /cache
-  Volume Name : cache-volume

---

`문제풀이`

https://v1-18.docs.kubernetes.io/docs/concepts/storage/volumes/#example-pod

```sh
kubectl create ns volume-1
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
  namespace: volume-1
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

<br/>
<br/>






### Secret를 만들고 첫번째 Pod의 특정 경로에 File 마운트, 두번째 Pod의 env에 매칭 
-  Secret : Name(mysecret), value(username:sec1) 
-  Pod1 : Name(mypod), Image(redis), path(/etc/foo)
-  Pod2 : Name(secret-env-pod), Image(redis), env(username->SECRET_USERNAME)

---

`문제풀이`

secret 만들기

https://v1-18.docs.kubernetes.io/docs/concepts/configuration/secret/#use-case-pods-with-prod-test-credentials


```sh
kubectl create secret generic mysecret --from-literal=username=sec1
```


<https://v1-18.docs.kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-files-from-a-pod>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
```

<https://v1-18.docs.kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
```


```sh
kubectl exec mypod -it -- ls /etc/foo
kubectl exec secret-env-pod -it -- env | grep SECRET_USERNAME
```


<br/>
<br/>







### Deployment 만든 후에 스펙을 저장하고 삭제하기
-  Replicas : 5
-  Image : redis
-  Label : test=deploy2

---

`문제풀이`

 
1.18 버전부터 kubectl run으로 Deployment을 만드는건 Deprecated 됐습니다

```sh
kubectl run deploy-2 --image=redis --replicas=5 --labels=test=deploy2
```

아래 URL을 참조해서 vi로 yaml 파일을 만드는 방식으로 교체합니다.

<https://v1-18.docs.kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment>

```sh
vi deploy-2.yaml
---------------------------- 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-2
  labels:
    test: deploy2
spec:
  replicas: 5
  selector:
    matchLabels:
      test: deploy2
  template:
    metadata:
      labels:
        test: deploy2
    spec:
      containers:
      - name: redis
        image: redis
        ports:
        - containerPort: 80
----------------------------
kubectl create deploy-2.yaml
```

```sh
kubectl get deployments.apps deploy-2 -o yaml > ./deploy-2.yaml
kubectl delete deployments.apps deploy-2
```


<br/>
<br/>


### 존재하는 Deployment에 scale을 5로 늘리기
- deployment name : scale-deployment

---

`사전환경`

```sh
cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: scale-deployment
  labels:
    test: scale
spec:
  replicas: 1
  selector:
    matchLabels:
      test: scale
  template:
    metadata:
      labels:
        test: scale
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
EOF
```


`문제풀이`

https://v1-18.docs.kubernetes.io/docs/reference/kubectl/cheatsheet/#scaling-resources

```sh 
kubectl scale deployment/scale-deployment --replicas=5
```


<br/>
<br/>


### Deployment 만들고 Upgrade 및 Rollback하기
- image (nginx:1.14.2) Replcas (3)
- 이미지를 1.16.1로 rolling-upgrade하고 record하기
- 이전 이미지로 rollback 하기


---

`문제풀이`


<br/>

1.18 버전부터 kubectl run으로 Deployment을 만드는건 Deprecated 됐습니다. 

```sh
kubectl run deploy-01 --image=nginx:1.14.2 --replicas=3
```

아래 URL을 참조해서 vi로 yaml 파일을 만드는 방식으로 교체합니다.

<https://v1-18.docs.kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment>

```sh
vi deploy-01.yaml
---------------------------- 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy-01
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
----------------------------
kubectl create deploy-01.yaml
```

```sh
kubectl set image deployment/deploy-01 nginx=nginx:1.16.1 --record
kubectl rollout undo deployment.v1.apps/deploy-01
```

ref.

```sh
kubectl rollout status deployment deploy-01
kubectl rollout history deployment deploy-01
```


<br/>
<br/>

### Deployment를 만들고 Service를 연결해서 nslookup로 Pod Dns와 Service Dns 조회한 내용을 저장
---


`문제풀이`


1.18 버전부터 kubectl run으로 Deployment을 만드는건 Deprecated 됐습니다. 

```sh
kubectl run dns-deploy --image=nginx --replicas=1
```

아래 URL을 참조해서 vi로 yaml 파일을 만드는 방식으로 교체합니다.

<https://v1-18.docs.kubernetes.io/docs/concepts/workloads/controllers/deployment/#creating-a-deployment>

```sh
vi dns-deploy.yaml
---------------------------- 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: dns-deploy
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
----------------------------
kubectl create dns-deploy.yaml
```

```sh 
kubectl expose deployment dns-deploy --port=8080 --name=dns-svc
```

https://v1-18.docs.kubernetes.io/ko/docs/concepts/services-networking/connect-applications-service/#dns

```sh
kubectl get pods dns-deploy-5d6bd489d5-n4sx4 -o wide
 
kubectl run curl1 --image=radial/busyboxplus:curl -i --tty
#nslookup 20.111.156.71
#nslookup dns-svc

kubectl delete deployments.apps curl
```

<br/>
<br/>







### DaemonSet을 이용해서 모든 노드에 nginx pod 생성, Taints 오버라이트 하지 말것
---


`문제풀이`

<https://v1-18.docs.kubernetes.io/docs/concepts/workloads/controllers/daemonset/>


```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ds-01
  labels:
    k8s-app: nginx
spec:
  selector:
    matchLabels:
      name: nginx
  template:
    metadata:
      labels:
        name: nginx
    spec:
      #tolerations:
      #- key: node-role.kubernetes.io/master
      #  effect: NoSchedule
      containers:
      - name: nginx
        image: nginx
```

```sh
kubectl apply -f daemon1.yaml
kubectl get pods
```


<br/>
<br/>

### ready 상태인 Node 갯수 저장하기, 단 Taints로 NoSchedule가 걸려 있는 node는 제외
---


`문제풀이`

```sh 
kubectl describe nodes | grep Taints
```

<br/>
<br/>

### CPU 부하가 가장 큰 Pod 이름 저장하기
- Label : cpu:high 

---


`문제풀이`

```sh 
kubectl top pods -l cpu=high
```

<br/>
