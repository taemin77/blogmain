
---
title: "Pod"
linkTitle: "Pod"
weight: 4
date: 2021-08-02
description: > 
  Container, Label, NodeSchedule
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/beginner/Object_Pod.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/beginner/Pod with Container, Label, Node Schedule for Kubernetes.jpg"
       alt="Pod with Container, Label, Node Schedule for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>



## 1. Container

---

<figure>
  <img src="/img/practice/beginner/Pod with Container Port for Kubernetes.jpg"
       alt="Pod with Container Port for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

### 1-1) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container1
    image: kubetm/p8000
    ports:
    - containerPort: 8000
  - name: container2
    image: kubetm/p8080
    ports:
    - containerPort: 8080
```

### 1-2) ReplicationController
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: replication-1
spec:
  replicas: 1
  selector:
    app: rc
  template:
    metadata:
      name: pod-1
      labels:
        app: rc
    spec:
      containers:
      - name: container
        image: kubetm/init
```

<br/>
<br/>


## 2. Label

---

<figure>
  <img src="/img/practice/beginner/Pod with Label Selector for Kubernetes.jpg"
       alt="Pod with Label Selector for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 2-1) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
  labels:
    type: web
    lo: dev
spec:
  containers:
  - name: container
    image: kubetm/init
```

### 2-2) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    type: web
  ports:
  - port: 8080
```

<br/>
<br/>


## 3. Node Schedule

---

<figure>
  <img src="/img/practice/beginner/Pod with Node Schedule for Kubernetes.jpg"
       alt="Pod with Node Schedule for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 3-1) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/init
```

### 3-2) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4
spec:
  containers:
  - name: container
    image: kubetm/init
    resources:
      requests:
        memory: 2Gi
      limits:
        memory: 3Gi
```

<br/>



{{% alert title="CPU와 Memory의 (request, limits)에 대한 강의 내용 보충" color="primary" %}}
CPU가 limits 수치까지 올라갔다고 해서 무조건 Request 수치까지 core를 낮추는 것이 아니고,
Node의 전체 부하 상태가 OverCommit된 상태일때 동작합니다.
Node 위의 Pod들이 Node의 자원을 모두 사용하고도 그 이상을 자원을 요구하게 되었을 때
Limit 수치까지 CPU를 사용하고 있는 Pod들에 한해서 그 수치를 Reqeust까지 떨어뜨리게 되며
Memory의 경우에도 그러한 상태일때 Limit까지 올라간 Pod들에 한해 재기동 시키게 됩니다. 
{{% /alert %}}

<br/>
<br/>



## Sample Yaml
---

### __Pod__

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-4                           # Pod 이름
  labels:                               # Label 
    type: web                           
    lo: dev  
spec:
  nodeSelector:                         # Node 직접 지정시
    kubernetes.io/hostname: k8s-node1   
  containers:
  - name: container                     # Container 이름
    image: kubetm/init                  # 이미지 선택
    ports:
    - containerPort: 8080               
    resources:                          # 자원 사용량 설정
      requests:
        memory: 1Gi
      limits:
        memory: 1Gi
```

<br/>
<br/>

## kubectl
---

### __Create__
``` sh
# 파일이 있을 경우
kubectl create -f ./pod.yaml

# 내용과 함께 바로 작성
kubectl create -f - <<END
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container
    image: kubetm/init
END
```

###  __Apply__
``` sh
kubectl apply -f ./pod.yaml
```

###  __Get__
``` sh
# 기본 Pod 리스트 조회 (Namepsace 포함)
kubectl get pods -n defalut

# 좀더 많은 내용 출력
kubectl get pods -o wide

# Pod 이름 지정
kubectl get pod pod1

# Json 형태로 출력
kubectl get pod pod1 -o json
```

### __Describe__
``` sh
# 상세 출력
kubectl describe pod pod1
```

### __Delete__
``` sh
# 파일이 있을 경우 생성한 방법 그대로 삭제
kubectl delete -f ./pod.yaml

# 내용과 함께 바로 작성한 경우 생성한 방법 그대로 삭제
kubectl delete -f - <<END
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  containers:
  - name: container
    image: kubetm/init
END

# Pod 이름 지정
kubectl delete pod pod1
```

###  __Exec__
``` sh
# Pod이름이 pod1인 Container로 들어가기 (나올땐 exit)
kubectl exec pod1 -it /bin/bash

# Container가 두개 이상 있을때 Container이름이 con1인 Container로 들어가기 
kubectl exec pod1 -c con1 -it /bin/bash
```

<br/>
<br/>

## Tips
---

###  __Apply vs Create__
  - 둘다 자원을 생성할때 사용할 수 있지만, [__Create__]는 기존에 같은 이름의 Pod가 존재하면 생성이 안되고, [__Apply__]는 기존에 같은 이름의 Pod가 존재하면 업데이트됨
  
<br/>
<br/>

## Referenece
---

### __Kubernetes__
  - __Pod Overview__ : <https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/>
  - __Labels and Selectors__ : <https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/>
  - __Assigning Pods to Nodes__ : <https://kubernetes.io/docs/concepts/configuration/assign-pod-node/>

