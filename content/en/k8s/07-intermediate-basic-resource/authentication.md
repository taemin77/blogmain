
---
title: "Authentication"
linkTitle: "Authentication"
weight: 3
date: 2021-08-02
description: > 
  X509 Certs, Kubectl, ServiceAccount
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/intermediate/Authentication.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/intermediate/Access API with Authenticaiton for Kubernetes.jpg"
       alt="Access API with Authenticaiton for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>

## 1. X509 Client Certs
---


<figure>
  <img src="/img/practice/intermediate/Access API with Authenticaiton X509 for Kubernetes.jpg"
       alt="Access API with Authenticaiton X509 for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

### 1-1) kubeconfig 인증서 확인

Path : /etc/kubernetes/admin.conf

```sh
cluster.certificate-authority-data : CA.crt (Base64)
user.client-certificate-data: Client.crt (Base64)
user.client-key-data: Client.key (Base64)
```

```sh
grep 'client-certificate-data' /etc/kubernetes/admin.conf | head -n 1 | awk '{print $2}' | base64 -d
grep 'client-key-data' /etc/kubernetes/admin.conf | head -n 1 | awk '{print $2}' | base64 -d
```

<br/>

### 1-2) Https API (Client.crt, Client.key)

case1) postman

```sh
https://192.168.0.30:6443/api/v1/nodes
```

```sh
Settings > General > SSL certificate verification > OFF
Settings > Certificates > Client Certificates > Host, CRT file, KEY file
```

case2) curl

```sh
curl -k --key ./Client.key --cert ./Client.crt https://192.168.0.30:6443/api/v1/nodes
```


### 1-3) kubectl config 세팅

kubeadm / kubectl / kubelet 설치

```sh
yum install -y --disableexcludes=kubernetes kubeadm-1.15.5-0.x86_64 kubectl-1.15.5-0.x86_64 kubelet-1.15.5-0.x86_64
```

admin.conf 인증서 복사

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Kubectl Proxy 띄우기

```sh
nohup kubectl proxy --port=8001 --address=192.168.0.30 --accept-hosts='^*$' >/dev/null 2>&1 &
```

### 1-4) HTTP API 호출 (Proxy)


case1) postman 

```sh
http://192.168.0.30:8001/api/v1/nodes
```

case2) curl

```sh
curl http://192.168.0.30:8001/api/v1/nodes
```

<br/>
<br/>


## 2. kubectl

---

<figure>
  <img src="/img/practice/intermediate/Access API with Authenticaiton kubectl for Kubernetes.jpg"
       alt="Access API with Authenticaiton kubectl for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 2-1) Cluster A kubeconfig
path : /etc/kubernetes/admin.conf


### 2-2) Cluster B kubeconfig
path : /etc/kubernetes/admin.conf


### 2-3) kubeconfig
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1KVEUtLS0tLQo=
    server: https://192.168.0.30:6443
  name: cluster-a
- cluster:
    certificate-authority-data: LS0tLS1KVEUtLS0tLQo=
    server: https://192.168.0.50:6443
  name: cluster-b
contexts:
- context:
    cluster: cluster-a
    user: admin-a
  name: context-a
- context:
    cluster: cluster-b
    user: admin-b
  name: context-b
current-context: context-a
kind: Config
preferences: {}
users:
- name: admin-a
  user:
    client-certificate-data: LS0tLS1KVEUtLS0tLQo=
    client-key-data: LS0tLS1KVEUtLS0tLQo=
- name: admin-b
  user:
    client-certificate-data: LS0tLS1KVEUtLS0tLQo=
    client-key-data: LS0tLS1KVEUtLS0tLQo=
```

### 2-4) kubectl CLI

kubectl download : 
https://kubernetes.io/docs/tasks/tools/install-kubectl/

windows

```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/windows/amd64/kubectl.exe
```

```sh
C:\Users\taemin
kubectl config use-context context-a
```

```sh
kubectl get nodes
```

<br/>
<br/>




## 3. Service Account

---

<figure>
  <img src="/img/practice/intermediate/Access API with Authenticaiton Service Account for Kubernetes.jpg"
       alt="Access API with Authenticaiton Service Account for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 3-1) Namespace
```sh
kubectl create ns nm-01
```

### 3-2) ServiceAccount & Secret 확인
```sh
kubectl describe -n nm-01 serviceaccounts
kubectl describe -n nm-01 secrets
```

### 3-3) Pod
```sh
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  namespace: nm-01
  labels:
     app: pod
spec:
  containers:
  - name: container
    image: kubetm/app
EOF
```


### 3-4) Https API 호출 (Token)

case1) http

`header` Authorization : Bearer TOKEN

```sh
https://192.168.0.30:6443/api/v1
https://192.168.0.30:6443/api/v1/namespaces/nm-01/pods/
```

case2) curl

```sh
curl -k -H "Authorization: Bearer TOKEN" https://192.168.0.30:6443/api/v1
curl -k -H "Authorization: Bearer TOKEN" https://192.168.0.30:6443/api/v1/namespaces/nm-01/pods/
```



<br/>
<br/>





## Referenece
---
### __Kubernetes__
  - __Authenticating__ : <https://kubernetes.io/docs/reference/access-authn-authz/authentication/>
