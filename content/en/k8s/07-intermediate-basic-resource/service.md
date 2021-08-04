
---
title: "Service"
linkTitle: "Service"
weight: 1
date: 2021-08-02
description: > 
  Headless, Endpoint, ExternalName
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/intermediate/Service-Headless_ExternalName.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/intermediate/Service with Headless, Endpoint, ExternalName for Kubernetes.jpg"
       alt="Service with Headless, Endpoint, ExternalName for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>


## Dns
<figure>
  <img src="/img/practice/intermediate/Service with DNS Concept for Kubernetes.jpg"
       alt="Service with DNS Concept for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

## Headless
<figure>
  <img src="/img/practice/intermediate/Service with Headless Concept for Kubernetes.jpg"
       alt="Service with Headless Concept for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

## Exteranl
<figure>
  <img src="/img/practice/intermediate/Service with ExternalName Concept for Kubernetes.jpg"
       alt="Service with ExternalName Concept for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

<br/>
<br/>


## Headless
---

<figure>
  <img src="/img/practice/intermediate/Service with DNS Practice for Kubernetes.jpg"
       alt="Service with DNS Practice for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 1-1) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip1
spec:
  selector:
    svc: clusterip
  ports:
  - port: 80
    targetPort: 8080
```

### 1-2) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    svc: clusterip
spec:
  containers:
  - name: container
    image: kubetm/app
```

### 1-3) Request Pod
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
kubectl exec request-pod -it /bin/bash
```

nslookup

```sh
nslookup clusterip1
nslookup clusterip1.default.svc.cluster.local
```

curl

```sh
curl clusterip1/hostname
curl clusterip1.default.svc.cluster.local/hostname
```

<br/>
<br/>

<figure>
  <img src="/img/practice/intermediate/Service with Headless Practice for Kubernetes.jpg"
       alt="Service with Headless Practice for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 2-1) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless1
spec:
  selector:
    svc: headless
  ports:
    - port: 80
      targetPort: 8080    
  clusterIP: None
```

### 2-2) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod4
  labels:
    svc: headless
spec:
  hostname: pod-a
  subdomain: headless1
  containers:
  - name: container
    image: kubetm/app
```

Nslookup


```sh
nslookup headless1
nslookup pod-a.headless1
nslookup pod-b.headless1

```
Curl

```sh
curl pod-a.headless1:8080/hostname
curl pod-b.headless1:8080/hostname
```

<br/>
<br/>


## Endpoint

---


<figure>
  <img src="/img/practice/intermediate/Service with Endpoint Practice1 for Kubernetes.jpg"
       alt="Service with Endpoint Practice1 for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 3-1) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: endpoint1
spec:
  selector:
    svc: endpoint
  ports:
  - port: 8080
```

### 3-2) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod7
  labels:
    svc: endpoint
spec:
  containers:
  - name: container
    image: kubetm/app
```


```sh
kubectl describe endpoints endpoint1
```

<br/>
<br/>


<figure>
  <img src="/img/practice/intermediate/Service with Endpoint Practice2 for Kubernetes.jpg"
       alt="Service with Endpoint Practice2 for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

### 4-1) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: endpoint2
spec:
  ports:
  - port: 8080
```

### 4-2) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod9
spec:
  containers:
  - name: container
    image: kubetm/app
```

### 4-3) Endpoint
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: endpoint2
subsets:
 - addresses:
   - ip: 20.109.5.12
   ports:
   - port: 8080
```

```sh
curl endpoint2:8080/hostname
```

<br/>
<br/>


<figure>
  <img src="/img/practice/intermediate/Service with Endpoint Practice3 for Kubernetes.jpg"
       alt="Service with Endpoint Practice3 for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

### 5-1) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: endpoint3
spec:
  ports:
  - port: 80
```

Github - Ip Address

```sh
nslookup https://www.github.com
curl -O 185.199.110.153:80/kubetm/kubetm.github.io/blob/master/sample/practice/intermediate/service-sample.md
```

### 5-2) Endpoint
```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: endpoint3
subsets:
 - addresses:
   - ip: 185.199.110.153
   ports:
   - port: 80
```

```sh
curl -O endpoint3/kubetm/kubetm.github.io/blob/master/sample/practice/intermediate/service-sample.md
```

<br/>
<br/>



## ExternalName

---

<figure>
  <img src="/img/practice/intermediate/Service with ExternalName Practice for Kubernetes.jpg"
       alt="Service with ExternalName Practice for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 6-1) Service
```yaml
apiVersion: v1
kind: Service
metadata:
 name: externalname1
spec:
 type: ExternalName
 externalName: github.github.io
```

```sh
curl -O externalname1/kubetm/kubetm.github.io/blob/master/sample/practice/intermediate/service-sample.md
```

<br/>
<br/>

## yaml 
---

### __Service__

```yaml
apiVersion: v1
kind: Service
metadata:
  name: headless1
spec:
  selector:             # 생략시 Endpoints 직접 생성해서 사용
    svc: headless
  ports:
    - port: 80
      targetPort: 8080    
  clusterIP: None       # headless 서비스  
  type: ExternalName    # ExternalName Service 설정시 사용
  externalName: github.github.io  # ExternalName사용시 연결 Domain지정
```

### __Endpoints__

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: headless1       # Service의 이름과 동일하게 지정
subsets:
 - addresses:
   - ip: 20.109.5.12    # Pod의 ClusterIp
   ports:
   - port: 8080         # Pod의 Container Port
```


### __Pod__

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod4
  labels:
    svc: headless
spec:
  hostname: pod-a       # 호스트네임 설정, 생략시 Pod Name이 적용됨
  subdomain: headless1  # headless 서비스 사용시 Service의 이름으로 지정
  containers:
  - name: container
    image: kubetm/app
```

<br/>
<br/>

## kubectl
---

### __Exec__

``` sh
# Pod이름이 request-pod인 Container로 들어가기 (나올땐 exit)
kubectl exec request-pod -it /bin/bash
```


### __Describe__
``` sh
# Endpoints 상세보기
kubectl describe endpoints endpoint1
```



<br/>
<br/>

## Tips
---
### __Dns__
  - Kubernetes 버전 1.11 이전의 Kubernetes DNS 서비스는 kube-dns를 기반
  - 버전 1.11은 kube-dns의 일부 보안 및 안정성 문제를 해결하기 위해 CoreDNS 를 도입

<br/>
<br/>

## Referenece
---
### __Kubernetes__
  - __DNS for Services and Pods__ : <https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/>
  - __Customizing DNS Service__ : <https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/>

### __Others__
  - __Kubernetes DNS-Based Service Discovery__ : <https://github.com/kubernetes/dns/blob/master/docs/specification.md>
  - __Kubernetes best practices: mapping external services__ : <https://cloud.google.com/blog/products/gcp/kubernetes-best-practices-mapping-external-services>
