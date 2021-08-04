
---
title: "Namespace, ResourceQuota, LimitRange"
linkTitle: "Namespace"
weight: 7
date: 2021-08-02
description: > 
  
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/beginner/Namespace_ResourceQuota_LimitRange.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/beginner/Namespace, ResourceQuota, LimitRange for Kubernetes.jpg"
       alt="Namespace, ResourceQuota, LimitRange for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>



## 1.  Namespace
---

<figure>
  <img src="/img/practice/beginner/How to use Namespace for Kubernetes.jpg"
       alt="How to use Namespace for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 1-1) Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-1
```

### 1-2) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-1
  labels:
    app: pod
spec:
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080
```

### 1-3) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
  namespace: nm-1
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
```

<br/>

### 1-1') Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-2
```

### 1-2') Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-2
  labels:
    app: pod
spec:
  containers:
  - name: container
    image: kubetm/init
    ports:
    - containerPort: 8080
```

<br/>

pod ip :

```sh
curl 10.16.36.115:8080/hostname
```
service ip :

```sh
curl 10.96.92.97:9000/hostname
```

<br/>


## Namespace Exception
---


### e-1) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30000
  type: NodePort
```

### e-2) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: pod-2
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: host-path
      mountPath: /mount1
  volumes:
  - name : host-path
    hostPath:
      path: /node-v
      type: DirectoryOrCreate
```


```sh
echo "hello" >> hello.txt
```

<br/>
<br/>



## 2. ResourceQuota
---

<figure>
  <img src="/img/practice/beginner/How to use ResourceQuota for Kubernetes.jpg"
       alt="How to use ResourceQuota for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

### 2-1) Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-3
```

### 2-2) ResourceQuota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: rq-1
  namespace: nm-3
spec:
  hard:
    requests.memory: 1Gi
    limits.memory: 1Gi
```

ResourceQuota Check Command

```sh
kubectl describe resourcequotas --namespace=nm-3
```

### 2-3) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
spec:
  containers:
  - name: container
    image: kubetm/app

```

### 2-4) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-3
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 0.5Gi
      limits:
        memory: 0.5Gi
```


<br/>
<br/>



## 3. LimitRange
---

<figure>
  <img src="/img/practice/beginner/How to use LimitRange1 for Kubernetes.jpg"
       alt="How to use LimitRange1 for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>




### 3-1) Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-5
```

### 3-2) LimitRange
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-1
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.4Gi
    maxLimitRequestRatio:
      memory: 3
    defaultRequest:
      memory: 0.1Gi
    default:
      memory: 0.2Gi
```

LimitRange Check Command

```sh
kubectl describe limitranges --namespace=nm-5
```

### 3-3) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: kubetm/app
    resources:
      requests:
        memory: 0.1Gi
      limits:
        memory: 0.5Gi
```

<br/>


## LimitRange Exception
---

### e-1) Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nm-6
```

### e-2) LimitRange
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-5
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.5Gi
    maxLimitRequestRatio:
      memory: 1
    defaultRequest:
      memory: 0.5Gi
    default:
      memory: 0.5Gi
```

### e-3) LimitRange
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: lr-3
spec:
  limits:
  - type: Container
    min:
      memory: 0.1Gi
    max:
      memory: 0.3Gi
    maxLimitRequestRatio:
      memory: 1
    defaultRequest:
      memory: 0.3Gi
    default:
      memory: 0.3Gi
```

### e-4) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: kubetm/app
```


<br/>


{{% alert title="LimitRange에 대한 강의 내용 보충" color="primary" %}}
ResourceQuota는 Namespace 뿐만 아니라 Cluster 전체에 부여할 수 있는 권한이지만,
LimitRange의 경우 Namespace내에서만 사용 가능합니다.
{{% /alert %}}

<br/>
<br/>


## kubectl
---


### __Describe__

``` sh
# nm-3의 Namespace에 있는 ResourceQuota들의 상세 조회
kubectl describe resourcequotas --namespace=nm-3
# nm-5의 Namespace에 있는 LimitRange들의 상세 조회
kubectl describe limitranges --namespace=nm-5
```


<br/>
<br/>




## Referenece
---

### * __Kubernetes__
  - __Share a Cluster with Namespaces__ : <https://kubernetes.io/docs/tasks/administer-cluster/namespaces/>
  - __Resource Quotas__ : <https://kubernetes.io/docs/concepts/policy/resource-quotas/>
  - __Limit Ranges__ : <https://kubernetes.io/docs/concepts/policy/limit-range/>