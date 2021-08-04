
---
title: "Statefulset"
linkTitle: "Statefulset"
weight: 1
date: 2021-08-02
description: > 
  Pod, PersistentVolume, Headless Service
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/intermediate/StatefulSet.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/intermediate/StatefulSet with satefull application for Kubernetes.jpg"
       alt="StatefulSet with satefull application for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>


<figure>
  <img src="/img/practice/intermediate/StatefulSet with Stateless, Stateful Application for Kubernetes.jpg"
       alt="StatefulSet with Stateless, Stateful Application for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



## 1. StatefulSet Controller

---

<figure>
  <img src="/img/practice/intermediate/StatefulSet with Pod for Kubernetes.jpg"
       alt="StatefulSet with Pod for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 1-1) ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-web
spec:
  replicas: 1
  selector:
    matchLabels:
      type: web
  template:
    metadata:
      labels:
        type: web
    spec:
      containers:
      - name: container
        image: kubetm/app
      terminationGracePeriodSeconds: 10
```

### 1-2) StatefulSet
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stateful-db
spec:
  replicas: 1
  selector:
    matchLabels:
      type: db
  template:
    metadata:
      labels:
        type: db
    spec:
      containers:
      - name: container
        image: kubetm/app
      terminationGracePeriodSeconds: 10
```



<br/>



## 2. PersistentVolumeClaim

---


<figure>
  <img src="/img/practice/intermediate/StatefulSet with PersistentVolumeClaims for Kubernetes.jpg"
       alt="StatefulSet with PersistentVolumeClaims for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 2-1) PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: replica-pvc1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: "fast"
```

### 2-2) ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replica-pvc
spec:
  replicas: 1
  selector:
    matchLabels:
      type: web2
  template:
    metadata:
      labels:
        type: web2
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-node1
      containers:
      - name: container
        image: kubetm/init
        volumeMounts:
        - name: storageos
          mountPath: /applog
      volumes:
      - name: storageos
        persistentVolumeClaim:
          claimName: replica-pvc1
      terminationGracePeriodSeconds: 10
```

```sh
touch /applog/server.log
```

### 2-3) StatefulSet
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: stateful-pvc
spec:
  replicas: 1
  selector:
    matchLabels:
      type: db2
  serviceName: "stateful-headless"
  template: 
    metadata:
      labels:
        type: db2
    spec:
      containers:
      - name: container
        image: kubetm/app
        volumeMounts:
        - name: volume
          mountPath: /applog
      terminationGracePeriodSeconds: 10
  volumeClaimTemplates:
  - metadata:
      name: volume
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 1G
      storageClassName: "fast"
```


<br/>



## 3. Headless Service

---

<figure>
  <img src="/img/practice/intermediate/StatefulSet with Headless Service for Kubernetes.jpg"
       alt="StatefulSet with Headless Service for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

### 3-1) Service (Headless)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: stateful-headless
spec:
  selector:
    type: db2
  ports:
    - port: 80
      targetPort: 8080    
  clusterIP: None
```

### 3-2) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: request-pod
spec:
  containers:
  - name: container
    image: kubetm/init
```


```sh
nslookup stateful-headless
curl stateful-pvc-0.stateful-headless:8080/hostname
```

<br/>
<br/>





## Referenece
---
### __Kubernetes__
  - __StatefulSets__ : <https://kubernetes.io/docs/concepts/workloads/controllers/statefulset>
  - __StatefulSet Basics__ : <https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set>