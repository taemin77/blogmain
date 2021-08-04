
---
title: "추가 기능"
linkTitle: "추가 기능"
weight: 2
date: 2021-08-02
description: >
  Helm Hook, Helm Test
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/helm/topic/Hook.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<br/><br/>



## 1. Helm Hook 기본 Flow

---

<figure>
  <img src="/img/helm/topic/Concept helm hook for Helm.jpg"
       alt="Concept helm hook for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>

### 1-1) Chart Template 생성 


```sh
helm create mychart
```
templates 폴더 안에 불필요한 파일 삭제 

```sh
rm -rf deployment.yaml  hpa.yaml  ingress.yaml  service.yaml  serviceaccount.yaml  tests/test-connection.yaml
```

### 1-2) pre-pod 생성

vi pre-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pre-pod
  annotations:
    helm.sh/hook: pre-upgrade
spec:
  restartPolicy: Never
  containers:
  - name: container
    image: kubetm/init
    command: ["sh", "-c", "echo 'start'; sleep 10; echo 'done'"]
```


### 1-3) Deployment 생성

vi deployment.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc
spec:
  selector:
    type: app
  ports:
  - port: 80
    targetPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
spec:
  selector:
    matchLabels:
      type: app
  replicas: 1
  template:
    metadata:
      labels:
        type: app
      annotations:
        rollme: {{ randAlphaNum 5 | quote }}
    spec:
      initContainers:
      - name: init-myservice
        image: kubetm/app
        command: ["sh", "-c", "echo 'start'; sleep 10; echo 'done'"]
      containers:
      - name: container
        image: kubetm/app
```

### 1-4) post-pod 생성

vi post-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: post-pod
  annotations:
    helm.sh/hook: post-upgrade
spec:
  restartPolicy: Never
  containers:
  - name: container
    image: kubetm/init
    command: ["sh", "-c", "echo 'start'; sleep 10; echo 'done'"]
```


### 1-5) test-pod 생성


mkdir tests
<br/>
vi test-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  annotations:
    helm.sh/hook: test
spec:
  restartPolicy: Never
  containers:
  - name: container
    image: kubetm/init
    command: ["sh", "-c", "echo 'start'; curl svc/hostname; echo 'done'"]
```


### 1-6) crd-pod.yaml 내용 업데이트 

mkdir crds
<br/>
vi crd-pod.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: crd-pod
  annotations:
    helm.sh/hook: pre-upgrade
spec:
  restartPolicy: Never
  containers:
  - name: container
    image: kubetm/init
    command: ["sh", "-c", "echo 'start'; sleep 10; echo 'done'"]
```



### 1-7) Install 명령


```sh
helm upgrade mychart ./../ -n nm-1 --create-namespace --install
kubectl get pods -n nm-1
```

### 1-8) Upgrade 명령


```sh
helm upgrade mychart ./../ -n nm-1 --create-namespace --install
kubectl get pods -n nm-1
```


### 1-9) Test 명령


```sh
helm test mychart -n nm-1
kubectl get pods -n nm-1
```
<br/>
<br/>





## 2. hook-weight

---

<figure>
  <img src="/img/helm/topic/hook-weight for Helm.jpg"
       alt="hook-weight for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

<br/>

### 2-1) Chart Template 생성 


```sh
helm create mychart2
```
templates 폴더 안에 불필요한 파일 삭제 

```sh
rm -rf deployment.yaml  hpa.yaml  ingress.yaml  service.yaml  serviceaccount.yaml  tests
```

### 2-2) pre-pod1 생성

vi pre-pod1.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pre-pod1
  annotations:
    helm.sh/hook: pre-install
    helm.sh/weight: "-1"
spec:
  restartPolicy: Never
  containers:
  - name: container
    image: kubetm/init
    command: ["sh", "-c", "echo 'start'; sleep 10; echo 'done'"]
```

### 2-3) pre-pod2 생성

vi pre-pod2.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pre-pod2
  annotations:
    helm.sh/hook: pre-install
spec:
  restartPolicy: Never
  containers:
  - name: container
    image: kubetm/init
    command: ["sh", "-c", "echo 'start'; sleep 10; echo 'done'"]
```

### 2-4) pre-pod3 생성

vi pre-pod3.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pre-pod3
  annotations:
    helm.sh/hook: pre-install
    helm.sh/hook-weight: "1"
spec:
  restartPolicy: Never
  containers:
  - name: container
    image: kubetm/init
    command: ["sh", "-c", "echo 'start'; sleep 10; echo 'done'"]
```

### 2-5) Deployment 생성

vi deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment
spec:
  selector:
    matchLabels:
      type: init
  replicas: 1
  template:
    metadata:
      labels:
        type: init
      annotations:
        rollme: {{ randAlphaNum 5 | quote }}
    spec:
      containers:
      - name: container
        image: kubetm/init
```

### 2-6) Helm Install 실행

```sh
helm upgrade mychart2 ./../ -n nm-2 --create-namespace --install
kubectl get pods -n nm-2
```


<br/>
<br/>


## 3. hook-delete-policy

---

<figure>
  <img src="/img/helm/topic/hook-delete-policy for Helm.jpg"
       alt="hook-delete-policy for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>

### 3-1) 이전 실습 리소스 제거


```sh
rm -rf pre-pod2.yaml pre-pod3.yaml
kubectl delete pods -n nm-2 pre-hook-pod2 pre-hook-pod3
kubectl get pods -n nm-2
```



### 3-2) pre-pod1 

vi pre-pod1.yaml

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pre-pod1
  annotations:
    helm.sh/hook: pre-upgrade
#   helm.sh/hook-delete-policy: hook-succeeded
#   helm.sh/hook-delete-policy: hook-failed
#   helm.sh/hook-delete-policy: before-hook-creation
spec:
  restartPolicy: Never
  containers:
  - name: container
    image: kubetm/init
    command: ["sh", "-c", "echo 'start'; sleep 10; echo 'done'"]
```

### 3-3) Helm Install 실행

```sh
helm upgrade mychart2 ./../ -n nm-2 --create-namespace --install
kubectl delete pods -n nm-2 pre-hook-pod1
kubectl get pods -n nm-2
```


<br/>
<br/>




## Referenece
---

### __Helm__
  - __Helm Hook__ : <https://helm.sh/docs/topics/charts_hooks/>
  - __Helm Test__ : <https://helm.sh/docs/topics/chart_tests/>
  