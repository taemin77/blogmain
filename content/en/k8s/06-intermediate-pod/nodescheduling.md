
---
title: "Node Scheduling"
linkTitle: "Scheduling"
weight: 1
date: 2021-08-02
description: > 
  Node Affinity, Pod Affinity/Anti-Affinity, Toleration/Taint
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/intermediate/Pod-Node_Scheduling.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>


<figure>
  <img src="/img/practice/intermediate/Scheduling with Affinity and Toleration Taint for Kubernetes.jpg"
       alt="Scheduling with Affinity and Toleration Taint for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>



## 1. Node Affinity

---

<figure>
  <img src="/img/practice/intermediate/Scheduling of Node Affinity for Kubernetes.jpg"
       alt="Scheduling of Node Affinity for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 1-1) Node Labeling

```sh
kubectl label nodes k8s-node1 kr=az-1
kubectl label nodes k8s-node2 us=az-1
```

### 1-2) MatchExpressions

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-match-expressions1
spec:
 affinity:
  nodeAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:   
    nodeSelectorTerms:
    - matchExpressions:
      -  {key: kr, operator: Exists}
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```


### 1-3) Required

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-required
spec:
 affinity:
  nodeAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchExpressions:
      - {key: ch, operator: Exists}
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```

### 1-4) Preferred

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-preferred
spec:
 affinity:
  nodeAffinity:
   preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 1
      preference:
       matchExpressions:
       - {key: ch, operator: Exists}
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```

<br/>


## 2. Pod Affinity / Anti-Affinity
---

<figure>
  <img src="/img/practice/intermediate/Scheduling of Pod Affinity for Kubernetes.jpg"
       alt="Scheduling of Pod Affinity for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 2-1) Node Labeling

```sh
kubectl label nodes k8s-node1 a-team=1
kubectl label nodes k8s-node2 a-team=2
```

### 2-2) Web1 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: web1
 labels:
  type: web1
spec:
 nodeSelector:
  a-team: '1'
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```

### 2-3) Web1 Affinity Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: server1
spec:
 affinity:
  podAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:   
   - topologyKey: a-team
     labelSelector:
      matchExpressions:
      -  {key: type, operator: In, values: [web1]}
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```

### 2-4) Web2 Affinity Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: server2
spec:
 affinity:
  podAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:   
   - topologyKey: a-team
     labelSelector:
      matchExpressions:
      -  {key: type, operator: In, values: [web2]}
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```

### 2-5) Web2 Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web2
  labels:
     type: web2
spec:
  nodeSelector:
    a-team: '2'
  containers:
  - name: container
    image: kubetm/app
  terminationGracePeriodSeconds: 0
```

<br/>

<figure>
  <img src="/img/practice/intermediate/Scheduling of Pod Anti-Affinity for Kubernetes.jpg"
       alt="Scheduling of Pod Anti-Affinity for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>

### 2-6) Master Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: master
  labels:
     type: master
spec:
  nodeSelector:
    a-team: '1'
  containers:
  - name: container
    image: kubetm/app
  terminationGracePeriodSeconds: 0
```

### 2-7) Master Anti-Affinity Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: slave
spec:
 affinity:
  podAntiAffinity:
   requiredDuringSchedulingIgnoredDuringExecution:   
   - topologyKey: a-team
     labelSelector:
      matchExpressions:
      -  {key: type, operator: In, values: [master]}
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```


<br/>


## 3. Taint / Toleration
---


<figure>
  <img src="/img/practice/intermediate/Scheduling of Taint and Toleration for Kubernetes.jpg"
       alt="Scheduling of Taint and Toleration for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 3-1) Node Labeling

```sh
kubectl label nodes k8s-node1 gpu=no1
```

### 3-2) Node1 - Taint

```yaml
kubectl taint nodes k8s-node1 hw=gpu:NoSchedule
```

### 3-3) Pod with Toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-with-toleration
spec:
 nodeSelector:
  gpu: no1
 tolerations:
 - effect: NoSchedule
   key: hw
   operator: Equal
   value: gpu
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```

### 3-4) Pod without Toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-without-toleration
spec:
 nodeSelector:
  gpu: no1
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```

### 3-5) Pod1 with NoExecute

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod1-with-no-execute
spec:
 tolerations:
 - effect: NoExecute
   key: out-of-disk
   operator: Exists
   tolerationSeconds: 30
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```

### 3-6) Pod2 with NoExecute

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod2-without-no-execute
spec:
 containers:
 - name: container
   image: kubetm/app
 terminationGracePeriodSeconds: 0
```

### 3-7) Node2 - Taint

```yaml
kubectl taint nodes k8s-node1 hw=gpu:NoSchedule-
kubectl taint nodes k8s-node2 out-of-disk=True:NoExecute
```

<br/>
<br/>


## Sample Yaml
---

### __NodeName__

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: node-name
spec:
 nodeName: k8s-node1
 containers:
 - name: container
   image: kubetm/app
```

<br/>

### __NodeSelector__

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: node-selector
spec:
 nodeSelector: 
  key1:value1
 containers:
 - name: container
   image: kubetm/app
```

<br/>

---
## kubectl
---

### __Label__
``` sh
# Node에 라벨 추가
kubectl label nodes k8s-node1 key1=value1
# Node에 라벨 삭제
kubectl label nodes k8s-node1 key1=value1-
```


<br/>
<br/>


## Tips
---


### NodeAffinity MatchExpressions Operator

<figure>
  <img src="/img/practice/intermediate/Scheduling MatchExpressions for Kubernetes.jpg"
       alt="Scheduling MatchExpressions for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


---
  
<br/>
<br/>


## Referenece
---

### __Kubernetes__
  - __Assigning Pods to Nodes__ : <https://kubernetes.io/docs/concepts/configuration/assign-pod-node>
  - __Taints and Tolerations__ : <https://kubernetes.io/docs/concepts/configuration/taint-and-toleration>

