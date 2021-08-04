
---
title: "Volume"
linkTitle: "Volume"
weight: 2
date: 2021-08-02
description: > 
  Dynamic Provisioning, StorageClass, Status, ReclaimPolicy
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/intermediate/Volume-Dynamic_Provisioning.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/intermediate/Volume with PV, PVC Concept for Kubernetes.jpg"
       alt="Volume with PV, PVC Concept for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>


## 1. 사전 구성 - StorageOS 설치 및 StorageClass 생성

---

<figure>
  <img src="/img/practice/intermediate/Volume with StorageOS Installation for Kubernetes.jpg"
       alt="Volume with StorageOS Installation for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 1-1) StorageOS Operator 설치

설치
```sh
kubectl apply -f https://github.com/storageos/cluster-operator/releases/download/1.5.0/storageos-operator.yaml
```

설치 확인

```sh
kubectl get all -n storageos-operator
```

Depolyment 수정

```sh
kubectl edit deployments.apps storageos-cluster-operator -n storageos-operator
```

spec.containers.env의 `DISABLE_SCHEDULER_WEBHOOK`의 Value를 `true`로 설정

```yaml
spec:
  containers:
  - command:
    - cluster-operator
    env:
    - name: DISABLE_SCHEDULER_WEBHOOK
      value: "false"    # true 로 변경
    image: storageos/cluster-operator:1.5.0
    imagePullPolicy: IfNotPresent
```

관리 계정을 위한 Secret 생성 (username 및 password를 Base64문자로 만들기)
 

```sh
echo -n "admin" | base64
echo -n "1234" | base64
```

`apiUsername` 및 `apiPassword` 부분에 위 결과로 나온 문자 넣기

```yaml
kubectl create -f - <<END
apiVersion: v1
kind: Secret
metadata:
  name: "storageos-api"
  namespace: "storageos-operator"
  labels:
    app: "storageos"
type: "kubernetes.io/storageos"
data:
  apiUsername: YWRtaW4=  # admin
  apiPassword: MTIzNA==  # 1234
END
```


<br/>

### 1-2) StorageOS 설치

StorageOS 설치 트리거 생성

```sh
kubectl apply -f - <<END
apiVersion: "storageos.com/v1"
kind: StorageOSCluster
metadata:
  name: "example-storageos"
  namespace: "storageos-operator"
spec:
  secretRefName: "storageos-api" # Reference the Secret created in the previous step
  secretRefNamespace: "storageos-operator"  # Namespace of the Secret
  k8sDistro: "kubernetes"
  images:
    nodeContainer: "storageos/node:1.5.0" # StorageOS version
  resources:
    requests:
    memory: "512Mi"
END
```



설치 확인


```sh
kubectl get all -n storageos
```


Dashboard 접속을 위한 Service 수정 (방법 1)

```sh
kubectl edit service storageos -n storageos
```
spec에  `externalIPs`와 Master IP 추가
```yaml
spec:
  clusterIP: 10.109.77.121
  externalIPs:     # 추가
  - 192.168.0.30   # Master IP 추가
  ports:
```

접속
```
http://192.168.0.30:5705/
```

<br/>

Dashboard 접속을 위한 Service 수정 (방법 2)

```sh
kubectl edit service storageos -n storageos
```
type을  `NodePort`로 변경
```yaml
spec:
  ports:
  - name: storageos
    port: 5705
    protocol: TCP
    targetPort: 5705
    nodePort: 30705  # port 번호 추가
  type: NodePort     # type 변경
```

접속
```
http://192.168.0.30:30705/
```
<br/>

### 1-3) Default StorageClass 추가

```sh
kubectl apply -f - <<END
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: default
  annotations: 
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/storageos
parameters:
  adminSecretName: storageos-api
  adminSecretNamespace: storageos-operator
  fsType: ext4
  pool: default
END
```

StorageClass 확인

```sh
kubectl get storageclasses.storage.k8s.io
```

```sh
NAME                PROVISIONER               AGE
default (default)   kubernetes.io/storageos   3s
fast                kubernetes.io/storageos   59s
```

<br/>
<br/>


## 2. Dynamic Provisioning
---

<figure>
  <img src="/img/practice/intermediate/Volume with Dynamic Provisioning Practice for Kubernetes.jpg"
       alt="Volume with Dynamic Provisioning Practice for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 2-1) PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-hostpath1
spec:
  capacity:
    storage: 1G
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /mnt/hostpath
    type: DirectoryOrCreate
```

### 2-2) PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-hostpath1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: ""
```


### 2-3) PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-fast1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: "fast"
```

### 2-4) PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-default1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 2G
```

<br/>
<br/>


## 3. PV Status, ReclaimPolicy
---

<figure>
  <img src="/img/practice/intermediate/Volume with PV Status, ReclaimPolicy Practice for Kubernetes.jpg"
       alt="Volume with PV Status, ReclaimPolicy Practice for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 3-1) Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-hostpath1
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  terminationGracePeriodSeconds: 0
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: hostpath
      mountPath: /mount1
  volumes:
  - name: hostpath
    persistentVolumeClaim:
      claimName: pvc-hostpath1
```

```sh
cd /mount1
touch file.txt
```

StorageOS Dashboard
```
http://192.168.0.30:5705/
```

<br/>

<figure>
  <img src="/img/practice/intermediate/Volume with ReclaimPolicy Recycle Practice for Kubernetes.jpg"
       alt="Volume with ReclaimPolicy Recycle Practice for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 3-2) PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-recycle1
spec:
  persistentVolumeReclaimPolicy: Recycle
  capacity:
    storage: 3G
  accessModes:
  - ReadWriteOnce
  hostPath:
    path: /tmp/recycle
    type: DirectoryOrCreate
```

### 3-3) PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-recycle1
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 3G
  storageClassName: ""
```

### 3-4) Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-recycle1
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  terminationGracePeriodSeconds: 0
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: hostpath
      mountPath: /mount1
  volumes:
  - name: hostpath
    persistentVolumeClaim:
      claimName: pvc-recycle1
```

```sh
cd /mount1
touch file.txt
```

<br/>
<br/>

## yaml 
---

### __StorageClass__

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: default
  annotations:
    # Default StorageClass로 선택 
    storageclass.kubernetes.io/is-default-class: "true" 
# 동적으로 PV생성시 PersistentVolumeReclaimPolicy 선택 (Default:Delete)
reclaimPolicy: Retain, Delete, Recycle
provisioner: kubernetes.io/storageos
# provisioner 종류에 따라 parameters의 하위 내용 다름 
parameters:                                             

```

<br/>
<br/>

## kubectl
---

### __Get All Objects in Namespaces__
``` sh
kubectl get all -n storageos-operator
```


### __Force Deletion__
``` sh
kubectl delete persistentvolumeclaims pvc-fast1 --namespace=default --grace-period 0 --force
kubectl delete persistentvolume pvc-b53fd802-3919-4fb0-8c1f-02221a3e4bc0 --grace-period 0 --force
```



<br/>
<br/>

## Tips
---
### __hostPath__
  - Recycle 정책은 /tmp/로 시작하는 Path에서만 됨


<br/>
<br/>

## Referenece
---
### __Kubernetes__
  - __StorageClass__ : <https://kubernetes.io/docs/concepts/storage/storage-classes>
  - __Dynamic Volume Provisioning__ : <https://kubernetes.io/docs/concepts/storage/dynamic-provisioning>

### __Others__
  - __StorageOS__ : <https://docs.storageos.com/docs/platforms/kubernetes/install/1.15>
  - __Ceph__ : <https://docs.ceph.com/docs/master/>
  - __Glusterfs__ : <https://github.com/gluster/gluster-kubernetes>
  - __Persistent Storage Strategies__ : <https://www.xenonstack.com/blog/persistent-storage>