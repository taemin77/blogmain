
---
title: "Service"
linkTitle: "Service"
weight: 5
date: 2021-08-02
description: > 
  ClusterIP, NodePort, LoadBalancer
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/beginner/Service.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/beginner/Service with ClusterIP, NodePort, LoadBalancer.jpg"
       alt="Service with ClusterIP, NodePort, LoadBalancer."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>


## 1. ClusterIP
---

<figure>
  <img src="/img/practice/beginner/Service with ClusterIP for Kubernetes.jpg"
       alt="Service with ClusterIP for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 1-1) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
     app: pod
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 8080
```

### 1-2) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
```

```sh
curl 10.104.103.107:9000/hostname
```

<br/>
<br/>


## 2. NodePort
---

<figure>
  <img src="/img/practice/beginner/Service with NodePort for Kubernetes.jpg"
       alt="Service with NodePort for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 2-1) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-2
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
    nodePort: 30000
  type: NodePort
  externalTrafficPolicy: Local
```


## 3. Load Balancer
---

<figure>
  <img src="/img/practice/beginner/Service with LoadBalancer for Kubernetes.jpg"
       alt="Service with LoadBalancer for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 3-1) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:
    app: pod
  ports:
  - port: 9000
    targetPort: 8080
  type: LoadBalancer
```

```sh
kubectl get service svc-3
```

<br/>
<br/>


## Sample Yaml
---

### __Service__

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-3
spec:
  selector:             # Pod의 Label과 매칭
    app: pod
  ports:
  - port: 9000          # Service 자체 Port
    targetPort: 8080    # Pod의 Container Port
  type: ClusterIP, NodePort, LoadBalancer  # 생략시 ClusterIP
  externalTrafficPolicy: Local, Cluster    # 트래픽 분배 역할
```

<br/>
<br/>


## kubectl
---

### __Get__

defalut 이름의 Namespace에서 svc-3 이름의 Service 조회

``` sh
kubectl get service svc-3 -n defalut
```

<br/>
<br/>


## Tips
---

###  __NodePort__
  - Node Port의 범위 : 30000~32767
  
<br/>
<br/>


## Referenece
---

###  __Kubernetes__
  - __Service__ : <https://kubernetes.io/docs/concepts/services-networking/service/>


###  __Others__
  - __Kubernetes NodePort vs LoadBalancer vs Ingress?__ : <https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0>

