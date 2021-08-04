
---
title: "ConfigMap, Secret"
linkTitle: "ConfigMap"
weight: 7
date: 2021-08-02
description: > 
  Env(Literal, File), Mount(File)
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/beginner/Configmapt_Secret.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/beginner/ConfigMap, Secret with Literal, File on Env, Mount for Kubernetes.jpg"
       alt="ConfigMap, Secret with Literal, File on Env, Mount for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>




## 1. Env (Literal)

---

<figure>
  <img src="/img/practice/beginner/ConfigMap, Secret with Literal for Kubernetes.jpg"
       alt="ConfigMap, Secret with Literal for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 1-1) ConfigMap
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-dev
data:
  SSH: 'false'
  User: dev
```

### 1-2) Secret
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: sec-dev
data:
  Key: MTIzNA==
```

### 1-3) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: container
    image: kubetm/init
    envFrom:
    - configMapRef:
        name: cm-dev
    - secretRef:
        name: sec-dev
```

<br/>
<br/>



## 2. Env (File)

---

<figure>
  <img src="/img/practice/beginner/ConfigMap, Secret with File for Kubernetes.jpg"
       alt="ConfigMap, Secret with File for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 2-1) Configmap

```sh
echo "Content" >> file-c.txt
kubectl create configmap cm-file --from-file=./file-c.txt
```

### 2-2) Secret

```sh
echo "Content" >> file-s.txt
kubectl create secret generic sec-file --from-file=./file-s.txt
```

### 2-3) Pod	
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-file
spec:
  containers:
  - name: container
    image: kubetm/init
    env:
    - name: file-c
      valueFrom:
        configMapKeyRef:
          name: cm-file
          key: file-c.txt
    - name: file-s
      valueFrom:
        secretKeyRef:
          name: sec-file
          key: file-s.txt
```

<br/>
<br/>



## 3. Volume Mount (File)

---


<figure>
  <img src="/img/practice/beginner/ConfigMap, Secret with Mount for Kubernetes.jpg"
       alt="ConfigMap, Secret with Mount for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 3-1) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-mount
spec:
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: file-volume
      mountPath: /mount
  volumes:
  - name: file-volume
    configMap:
      name: cm-file
```


<br/>
<br/>


## kubectl
---


### __ConfigMap__

``` sh
# file-c.txt 라는 파일로 cm-file라는 이름의 ConfigMap 생성
kubectl create configmap cm-file --from-file=./file-c.txt
# key1:value1 라는 상수로 cm-file라는 이름의 ConfigMap 생성
kubectl create configmap cm-file --from-literal=key1=value1
# 여러 key:value로 cm-file라는 이름의 ConfigMap 생성 
kubectl create configmap cm-file --from-literal=key1=value1 --from-literal=key2=value2
```

###  __Secret Generic__
``` sh
# file-s.txt 라는 파일로 sec-file라는 이름의 Secret 생성
kubectl create secret generic sec-file --from-file=./file-s.txt
# key1:value1 라는 상수로 sec-file라는 이름의 Secret 생성
kubectl create secret generic sec-file --from-literal=key1=value1
```

<br/>
<br/>


## Tips
---

### __Secret__
  - 데이터가 메모리에 저장되기 때문에 보안에 유리
  - 한 Secret당 최대 1M까지만 저장됨
  
<br/>
<br/>


## Referenece
---

### __Kubernetes__
  - __Configure a Pod to Use a ConfigMap__ : <https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/>
  - __Distribute Credentials Securely Using Secrets__ : <https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/>
