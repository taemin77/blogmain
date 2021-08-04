
---
title: "Storage Architecture"
linkTitle: "Storage"
weight: 3
date: 2021-08-02
description: > 
  FileStorage(NFS), BlockStorage(Longhorn)
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/architecture/Networking.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>


<figure>
  <img src="/img/practice/architecture/Storage Architecture with FileStorage, BlockStorage, ObjectStorage for Kubernetes.jpg"
       alt="Storage Architecture with FileStorage, BlockStorage, ObjectStorage for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>



## 1. FTP Server 구축
---


### 1-1) NFS 패키지 다운 및 설치

설치
```sh
yum -y install nfs-utils rpcbind
systemctl start rpcbind
systemctl start nfs-server
systemctl start rpc-statd
systemctl enable rpcbind
systemctl enable nfs-server
```

확인
```sh
systemctl status nfs-server
```

<br/>

### 1-2) 공유 폴더 생성 및 설정

공유 폴더 생성
```sh
mkdir /share-data
chmod 777 /share-data
```

`vi /etc/exports` 로 아래 내용 입력 후 저장
```sh
/share-data *(rw,sync,no_root_squash)
```

반영
```sh
exportfs -r
```

방화벽 해제 및 NFS Server재시작
```sh
systemctl stop firewalld && systemctl disable firewalld
systemctl stop NetworkManager && systemctl disable NetworkManager
systemctl restart nfs-server
```


node1 및 node2에 NFS 클라이언트 설치

```
sudo yum install nfs-utils
```

<br/>
<br/>	




## 2. FileStroage (NFS) 연결 Pod

---

<figure>
  <img src="/img/practice/architecture/NFS with FileStorage for Kubernetes.jpg"
       alt="NFS with FileStorage for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 2-1) NFS 연결을 위한 PersistentVolume 생성

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
  labels:
    pv: pv-nfs
spec:
  capacity:
    storage: 2G
  accessModes:
  - ReadWriteMany
  nfs:
    path: /share-data
    server: 192.168.219.10
```


<br/>


### 2-2) PersistentVolumeClaim 
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 2G
  storageClassName: ""
  selector:
    matchLabels:
      pv: pv-nfs
```

<br/>


### 2-3) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nfs
spec:
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: volume-nfs
      mountPath: /nfs/share-data
  volumes:
  - name : volume-nfs
    persistentVolumeClaim:
      claimName: pvc-nfs
```

<br/>
<br/>



## 3. Longhorn 구축
---


### 3-1) 모든 Master/Work Node에 iscsi 설치
```sh
yum install -y iscsi-initiator-utils
```

<br/>

### 3-2) Longhorn 설치
```sh
kubectl apply -f https://raw.githubusercontent.com/longhorn/longhorn/master/deploy/longhorn.yaml
```
<br/>

### 3-3) 확인

```sh
kubectl get pods -n longhorn-system
```

<br/>


### 3-4) 기존 Longhorn StorageClass 삭제 및 재생성
```sh
kubectl get storageclasses.storage.k8s.io -n longhorn-system longhorn
kubectl delete storageclasses.storage.k8s.io -n longhorn-system longhorn
```

```yaml
cat <<EOF | kubectl create -f -
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: longhorn
provisioner: driver.longhorn.io
allowVolumeExpansion: true
parameters:
  numberOfReplicas: "2"
  staleReplicaTimeout: "2880"
  fromBackup: ""
EOF
```

<br/>

### 3-5) Longhorn Dashboard 접속을 위한 Port Type변경 

Service Type 변경 [ClusterIP -> NodePort]
 
```sh
kubectl edit svc -n longhorn-system longhorn-frontend
```

<br/>
<br/>


## 4. Longhorn Volume에 POD 연결
---


<figure>
  <img src="/img/practice/architecture/Longhorn with BlockStorage for Kubernetes.jpg"
       alt="Longhorn with BlockStorage for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>




### 4-1) PersistentVolumeClaim 
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: longhorn-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: longhorn
  resources:
    requests:
      storage: 1Gi
```

<br/>

### 4-2) Longhorn Dashboard 접속 


```sh
http://localhost:30001
```

<br/>


### 4-3) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-blockstorage
spec:
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: volume-blockstorage
      mountPath: /longhorn/data
  volumes:
  - name : volume-blockstorage
    persistentVolumeClaim:
      claimName: longhorn-pvc
```

<br/>



### 4-4) Longhorn Volumeattachments 확인 
```sh
kubectl get -n longhorn-system volumeattachments.storage.k8s.io
```

<br/>


## Referenece
---

### __Kubernetes__
  - __PV Access Mode__ : <https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes>
  - __PV Provisioner__ : <https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner>
  - __CSI Developer Documentation__ : <https://kubernetes-csi.github.io/docs/>
  
### __Others__
  - __Install Longhorn__ : <https://longhorn.io/docs/0.8.1/deploy/install/install-with-kubectl/>

  
  