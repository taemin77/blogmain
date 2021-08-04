
---
title: "Deployment"
linkTitle: "Deployment"
weight: 11
date: 2018-07-3
description: > 
  Recreate, RollingUpdate
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/beginner/Deployment.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/beginner/Deployment with ReCreate, RollingUpdate for Kubernetes.jpg"
       alt="Deployment with ReCreate, RollingUpdate for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>



## 1. ReCreate

---

<figure>
  <img src="/img/practice/beginner/Deployment with ReCreate for Kubernetes.jpg"
       alt="Deployment with ReCreate for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>
 

### 1-1) Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-1
spec:
  selector:
    matchLabels:
      type: app
  replicas: 2
  strategy:
    type: Recreate
  revisionHistoryLimit: 1
  template:
    metadata:
      labels:
        type: app
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 10
```

<br/>
 

### 1-2) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    type: app
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080

```

### Command
```sh
while true; do curl 10.99.5.3:8080/version; sleep 1; done
```

### Kubectl
```sh
kubectl rollout undo deployment deployment-1 --to-revision=2
kubectl rollout history deployment deployment-1
```


<br/>
<br/>



## 2. RollingUpdate

---

<figure>
  <img src="/img/practice/beginner/Deployment with RollingUpdate for Kubernetes.jpg"
       alt="Deployment with RollingUpdate for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 2-1) Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-2
spec:
  selector:
    matchLabels:
      type: app2
  replicas: 2
  strategy:
    type: RollingUpdate
  minReadySeconds: 10
  template:
    metadata:
      labels:
        type: app2
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```

### 2-2) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  selector:
    type: app2
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```

### Command
```sh
while true; do curl 10.99.5.3:8080/version; sleep 1; done
```

<br/>
<br/>



## 3. Blue/Green

---


### ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica1
spec:
  replicas: 2
  selector:
    matchLabels:
      ver: v1
  template:
    metadata:
      name: pod1
      labels:
        ver: v1
    spec:
      containers:
      - name: container
        image: kubetm/app:v1
      terminationGracePeriodSeconds: 0
```

### Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:
    ver: v1
  ports:
  - port: 8080
    protocol: TCP
    targetPort: 8080
```

<br/>
<br/>


## Referenece
---

### __Kubernetes__
  - __Deployments__ : <https://kubernetes.io/docs/concepts/workloads/controllers/deployment/>