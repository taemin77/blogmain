
---
title: "차트 개발 팁과 요령"
linkTitle: "개발팁"
weight: 1
date: 2021-08-02
description: >
  Install, Upgrade, Uninstall
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/helm/topic/Tip.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<br/><br/>



## 1. Install 및 Upgrade Tip

---

<figure>
  <img src="/img/helm/topic/Install and Upgrade  for Helm.jpg"
       alt="Install and Upgrade  for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>

### 1-1) Chart Template 생성 


```sh
helm create mychart
```
templates 폴더 안에 불필요한 파일 삭제 

```sh
rm -rf deployment.yaml  hpa.yaml  ingress.yaml  service.yaml  serviceaccount.yaml  tests
```


### 1-2) deployment.yaml 내용 업데이트 

vi deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test
spec:
  selector:
    matchLabels:
      type: app
  replicas: 1
  template:
    metadata:
      labels:
        type: app
    spec:
      initContainers:
      - name: init-myservice
        image: kubetm/app
        command: ["sh", "-c", "echo 'start'; sleep 30; echo 'done'"]
      containers:
      - name: container
        image: kubetm/app
        envFrom:
        - configMapRef:
            name: test-cm
        volumeMounts:
        - name: volume
          mountPath: /hostpath
      volumes:
      - name : volume
        persistentVolumeClaim:
          claimName: test-pvc
```

### 1-3) configmap.yaml 추가

vi configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-cm
data:
  env: prod
  log: Error
```

### 1-4) pvc.yaml 추가

vi pvc.yaml

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: test-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: ""
  selector:
    matchLabels:
      name: test-pv
```

### 1-5) pv.yaml 추가

vi pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-pv
  labels:
    name: test-pv
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1G
  hostPath:
    path: /host1
    type: DirectoryOrCreate
```


### 1-6) Install, Upgrade 명령

```sh
cd ..
helm install mychart . -n nm-1

helm install mychart . -n nm-1 --create-namespace
kubectl get ns nm-1
kubectl get -n nm-1 pods

helm upgrade mychart . -n nm-1 
helm uninstall mychart -n nm-1 

helm upgrade mychart . -n nm-1 --create-namespace --install
helm upgrade mychart . -n nm-1 --create-namespace --install --wait --timeout 10m
```


<br/>
<br/>




## 2. Pod 자동 재기동 Tip

---

<figure>
  <img src="/img/helm/topic/Pod Auto Deployment for Helm.jpg"
       alt="Pod Auto Deployment for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>

### 2-1) configmap.yaml 수정

vi configmap.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-cm
data:
  env: prod
  log: Info
```

```sh
helm upgrade mychart . -n nm-1 --create-namespace --install
kubectl get -n nm-1 pods
kubectl get -n nm-1 cm -o yaml
kubectl exec -n nm-1 test-7c4fff79cd-4m5st -it env
```

### 2-2) deployment.yaml 내용 업데이트 

vi deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test
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
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
#        rollme: {{ randAlphaNum 5 | quote }}
    spec:
      containers:
      - name: container
      ...
```

```sh
helm upgrade mychart . -n nm-1 --create-namespace --install
kubectl get -n nm-1 pods
kubectl exec -n nm-1 test-7c4fff79cd-4m5st -it env

helm upgrade mychart . -n nm-1 --create-namespace --install
kubectl get -n nm-1 pods
kubectl get -n nm-1 deployment -o yaml
```


<br/>
<br/>




## 3. 기타 Resource 특성

---

<figure>
  <img src="/img/helm/topic/Etc Resource for Helm.jpg"
       alt="Etc Resource for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



<br/>

### 3-1) pv.yaml 내용 업데이트 

vi pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: test-pv
  labels:
    name: test-pv
spec:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 1G
  hostPath:
    path: /host2
    type: DirectoryOrCreate
```


```sh
helm upgrade mychart . -n nm-1 --create-namespace --install
helm uninstall mychart -n nm-1
kubectl get pv
kubectl get pvc -n nm-1
helm upgrade mychart . -n nm-1 --create-namespace --install
kubectl get pv -o yaml

helm list -n nm-1
helm uninstall mychart -n nm-1
helm list -n nm-1
kubectl get ns nm-1
kubectl delete ns nm-1
```

### 3-2) Uninstall시 Namespace 확인 

```sh
helm uninstall mychart -n nm-1
```

<br/>
<br/>




## 4. Helm 저장소

---

<figure>
  <img src="/img/helm/topic/Helm Deployment Repository for Helm.jpg"
       alt="Helm Deployment Repository for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>

### 4-1) Install 및 Upgrade 실행

```sh
helm upgrade mychart . -n nm-1 --create-namespace --install
```

### 4-2) Secret 확인

```sh
kubectl get -n nm-1 secret -l name=mychart
kubectl get -n nm-1 secret sh.helm.release.v1.mychart.v1 -o yaml
```

### 4-3) Helm History 확인

```sh
helm history -n nm-1 mychart
helm upgrade mychart . -n nm-1 --create-namespace --install --history-max 2
kubectl get -n nm-1 secret -l name=mychart
helm history -n nm-1 mychart
```


<br/>
<br/>




## Referenece
---

### __Helm__
  - __Helm Upgrade__ : <https://helm.sh/docs/helm/helm_upgrade/>
  - __Helm Install__ : <https://helm.sh/docs/helm/helm_install/>
  - __Helm Uninstall__ : <https://helm.sh/docs/helm/helm_uninstall/>
  - __Helm History__ : <https://helm.sh/docs/helm/helm_history/>
  - __Chart Development Tips and Tricks__ : <https://helm.sh/docs/howto/charts_tips_and_tricks/>
    