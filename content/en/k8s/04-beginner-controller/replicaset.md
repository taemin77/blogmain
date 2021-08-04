
---
title: "Replication Controller, ReplicaSet"
linkTitle: "ReplicaSet"
weight: 10
date: 2018-07-3
description: > 
  Template, Replicas, Selector
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/beginner/ReplicaSet.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/beginner/Controller with Replicastion Controller, ReplicaSet for Kubernetes.jpg"
       alt="Controller with Replicastion Controller, ReplicaSet for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>





## 1. Template, Replicas
 
---


<figure>
  <img src="/img/practice/beginner/Controller with Replication, ReplicaSet for Kubernetes.jpg"
       alt="Controller with Replication, ReplicaSet for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 1-1) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    type: web
spec:
  containers:
  - name: container
    image: kubetm/app:v1
  terminationGracePeriodSeconds: 0
```

<br/>

### 1-2) ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica1
spec:
  replicas: 1
  selector:
    matchLabels:
      type: web
  template:
    metadata:
      name: pod1
      labels:
        type: web
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```


<br/>
<br/>



## 2. Updating Controller

---

ReplicationController -> ReplicaSet

### ReplicationController
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: replication1
spec:
  replicas: 2
  selector:
    cascade: "false"
  template:
    metadata:
      labels:
        cascade: "false"
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
```

### kubectl
```sh
kubectl delete replicationcontrollers replication1 --cascade=false
```

### ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica2
spec:
  replicas: 2
  selector:
    matchLabels:
      cascade: "false"
  template:
    metadata:
      labels:
        cascade: "false"
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
```


<br/>
<br/>



## 3. Selector

---


<figure>
  <img src="/img/practice/beginner/Match with Replication, ReplicaSet for Kubernetes.jpg"
       alt="Match with Replication, ReplicaSet for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 3-1) ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica1
spec:
  replicas: 1
  selector:
    matchLabels:
      type: web
      ver: v1
    matchExpressions:
    - {key: type, operator: In, values: [web]}
    - {key: ver, operator: Exists}
  template:
    metadata:
      labels:
        type: web
        ver: v1
        location: dev
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```

<br/>
<br/>

### Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-node-affinity1
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIngnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
  	       - {key: AZ-01, operator: Exists}
  containers:
  - name: container
    image: kubetm/init
```


## Tip
---
###  MatchExpressions

<figure>
  <img src="/img/practice/beginner/Match2 with Replication, ReplicaSet for Kubernetes.jpg"
       alt="Match2 with Replication, ReplicaSet for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



<br/>
<br/>


## Referenece
---

###  __Kubernetes__
  - __ReplicaSet__ : <https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/>
  - __Labels and Selectors__ : <https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/>
  - __ReplicationController__ : <https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/>


