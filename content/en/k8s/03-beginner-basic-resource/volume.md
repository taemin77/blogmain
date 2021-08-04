
---
title: "Volume"
linkTitle: "Volume"
weight: 6
date: 2021-08-02
description: > 
  emptyDir, hostPath, PV/PVC
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/beginner/Volume.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>


<figure>
  <img src="/img/practice/beginner/Volume with emptyDir, hostPath, PV, PVC, for Kubernetes.jpg"
       alt="Volume with emptyDir, hostPath, PV, PVC, for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>



## 1. emptyDir
---


<figure>
  <img src="/img/practice/beginner/Volume with emptyDir for Kubernetes.jpg"
       alt="Volume with emptyDir for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

### 1-1) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-1
spec:
  containers:
  - name: container1
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount1
  - name: container2
    image: kubetm/init
    volumeMounts:
    - name: empty-dir
      mountPath: /mount2
  volumes:
  - name : empty-dir
    emptyDir: {}
```

```sh
mount | grep mount1
echo "file context" >> file.txt
```

<br/>
<br/>


## 2. hostPath
---

<figure>
  <img src="/img/practice/beginner/Volume with hostPath for Kubernetes.jpg"
       alt="Volume with hostPath for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 2-1) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-3
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

<br/>
<br/>


## 3. PVC / PV
---

<figure>
  <img src="/img/practice/beginner/Volume with PersistentVolume PersistentVolumeClaim for Kubernetes.jpg"
       alt="Volume with PersistentVolume PersistentVolumeClaim for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 3-1) PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-03
spec:
  capacity:
    storage: 2G
  accessModes:
  - ReadWriteOnce
  local:
    path: /node-v
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [k8s-node1]}
```

### 3-2) PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-04
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: ""
```

### 3-3) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-volume-3
spec:
  containers:
  - name: container
    image: kubetm/init
    volumeMounts:
    - name: pvc-pv
      mountPath: /mount3
  volumes:
  - name : pvc-pv
    persistentVolumeClaim:
      claimName: pvc-01
```

<br/>
<br/>



{{% alert title="PV-PVC를 label과 selector를 이용해 연결하는 방법 예시" color="primary" %}}

{{% /alert %}}



### PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-03
  labels:
    pv: pv-03
spec:
  capacity:
    storage: 2G
  accessModes:
  - ReadWriteOnce
  local:
    path: /node-v
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [k8s-node1]}
```

### PersistentVolumeClaim
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-04
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: ""
  selector:
    matchLabels:
      pv: pv-03
```

<br/>
<br/>

## Tips
---
###  __hostPath Type__
  - DirectoryOrCreate : 실제 경로가 없다면 생성
  - Directory : 실제 경로가 있어야됨
  - FileOrCreate : 실제 경로에 파일이 없다면 생성
  - File : 실제 파일이 었어야함

<br/>
<br/>

## Referenece
---
###  __Kubernetes__
  - __Volumes__ : <https://kubernetes.io/docs/concepts/storage/volumes/>



