
---
title: "AutoScaler - HPA"
linkTitle: "HPA"
weight: 3
date: 2021-08-02
description: > 
  HPA, VPA, CA
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/intermediate/Autoscaler.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/intermediate/Autoscaler with Horizontal Pod Autoscaler for Kubernetes.jpg"
       alt="Autoscaler with Horizontal Pod Autoscaler for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>

## 1. Metrics Server 설치

---


### 1-1) Metrics Server 다운 및 설치

설치
```sh
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.6/components.yaml
```

<br/>
Metrics Deployment 수정 (args 에 `kubelet-insecure-tls` 와 `kubelet-preferred-address-types=InternalIP` 추가)
```sh
kubectl edit deployment metrics-server -n kube-system

------------------------
spec:
   containers:
   - args:
     - --cert-dir=/tmp
     - --secure-port=4443
     - --kubelet-insecure-tls
     - --kubelet-preferred-address-types=InternalIP
     image: k8s.gcr.io/metrics-server-amd64:v0.3.6
     imagePullPolicy: IfNotPresent
     name: metrics-server
     ------------------------
```

<br/>
설치 확인 (True값 확인)

```sh
kubectl get apiservices |egrep metrics

------------------------
v1beta1.metrics.k8s.io                     kube-system/metrics-server   True        28m
------------------------
```

<br/>
메트릭 값 확인 (1~2분 후)

```sh
kubectl top node

------------------------
NAME             CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
k8s-master       485m         9%     4852Mi          32%       
k8s-node1        413m         8%     4929Mi          33%       
k8s-node2        554m         11%    4672Mi          31%       
------------------------
```



<br/>
<br/>




## 2. HPA (Horizontal Pod Autoscaler)

---


<figure>
  <img src="/img/practice/intermediate/Autoscaler with Horizontal Pod Autoscaler CPU for Kubernetes.jpg"
       alt="Autoscaler with Horizontal Pod Autoscaler CPU for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 2-1) Target Deployment (CPU) / Service
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: stateless-cpu1
spec:
 selector:
   matchLabels:
      resource: cpu
 replicas: 2
 template:
   metadata:
     labels:
       resource: cpu
   spec:
     containers:
     - name: container
       image: kubetm/app:v1
       resources:
         requests:
           cpu: 10m
         limits:
           cpu: 20m
---
apiVersion: v1
kind: Service
metadata:
 name: stateless-svc1
spec:
 selector:
    resource: cpu
 ports:
   - port: 8080
     targetPort: 8080
     nodePort: 30001
 type: NodePort
```

<br/>

### 2-2) HPA - Resource (Utilization)
```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-resource-cpu
spec:
  maxReplicas: 10
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: stateless-cpu1
  metrics:
  - type: Resource 
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

```sh
kubectl get hpa -w
```

```sh
while true;do curl 192.168.0.30:30001/hostname; sleep 0.01; done
```

```sh
kubectl delete horizontalpodautoscalers.autoscaling hpa-resource-cpu
```

<br/>
<br/>

<figure>
  <img src="/img/practice/intermediate/Autoscaler with Horizontal Pod Autoscaler Memory for Kubernetes.jpg"
       alt="Autoscaler with Horizontal Pod Autoscaler Memory for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 2-3) Target Deployment (Memory) / Service
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: stateless-memory1
spec:
 selector:
   matchLabels:
      resource: memory
 replicas: 2
 template:
   metadata:
     labels:
       resource: memory
   spec:
     containers:
     - name: container
       image: kubetm/app:v1
       resources:
         requests:
           memory: 10Mi
         limits:
           memory: 20Mi
---
apiVersion: v1
kind: Service
metadata:
 name: stateless-svc2
spec:
 selector:
    resource: memory
 ports:
   - port: 8080
     targetPort: 8080
     nodePort: 30002
 type: NodePort
```

<br/>

### 2-4) HPA - Resource (AverageValue)

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-resource-memory
spec:
  maxReplicas: 10
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: stateless-memory1
  metrics:
  - type: Resource 
    resource:
      name: memory
      target:
        type: AverageValue
        averageValue: 5Mi
```

```sh
while true;do curl 192.168.0.30:30002/hostname; sleep 0.01; done
```

```sh
kubectl delete horizontalpodautoscalers.autoscaling hpa-resource-memory
```

<br/>
<br/>


## Sample yaml 
---

### * __HPA for - Pods__

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-pods
spec:
  maxReplicas: 10
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: stateless-app1
  metrics:
  - type: Pods
    pods:
      metric:
        name: packets-per-second
      target:
        type: AverageValue
        averageValue: 10
```


### __HPA - Object__

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa-object
spec:
  maxReplicas: 10
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: stateless-app1
  metrics:
  - type: Object
    object:
      metric:
        name: requests-per-second
      target:
        type: Value
        value: 10
      describedObject:
        apiVersion: networking.k8s.io/v1beta1
        kind: Ingress
        name: ingress-hpa
```

<br/>
<br/>


## Referenece
---

### __Kubernetes__
  - __Horizontal Pod Autoscaler__ : <https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/>
  - __Horizontal Pod Autoscaler Walkthrough__ : <https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/>

### __Others__
  - __Best Practices for Cluster Autoscaler, HPA and VPA__ : <https://www.replex.io/blog/kubernetes-in-production-best-practices-for-cluster-autoscaler-hpa-and-vpa>
  - __Cluster Autoscaler, Horizontal Pod Autoscaler, and Vertical Pod Autoscaler__ : <https://levelup.gitconnected.com/kubernetes-autoscaling-101-cluster-autoscaler-horizontal-pod-autoscaler-and-vertical-pod-2a441d9ad231>
  - __How to autoscale apps on Kubernetes with custom metrics__ : <https://learnk8s.io/autoscaling-apps-kubernetes>



