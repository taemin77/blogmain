
---
title: "Dashboard"
linkTitle: "Dashboard"
weight: 5
date: 2021-08-02
description: > 
  Kubeconfig, Token
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/intermediate/Kubernetes_Dashboard.pdf" download>
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


## 1. Dashboard 2.0.0 설치
---

<figure>
  <img src="/img/practice/intermediate/Access API with Dashaboard 2.0.0 for Kubernetes.jpg"
       alt="Access API with Dashaboard 2.0.0 for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

<br/>

### 1-1)  Dashboard 설치 

kubetm 가이드로 Dashboard(1.10.1)를 설치했을 경우 아래 명령으로 삭제

```sh
kubectl delete -f https://raw.githubusercontent.com/kubetm/kubetm.github.io/master/sample/practice/appendix/gcp-kubernetes-dashboard.yaml
```

새 Dashboard (2.0.0) 설치 - <https://github.com/kubernetes/dashboard>

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```


<br/>

### 1-2) ClusterRoleBinding 생성

```sh
cat <<EOF | kubectl create -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard2
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
EOF
```

<br/>

### 1-3) Token 확인

```sh
kubectl -n kubernetes-dashboard get secret kubernetes-dashboard-token- \-o jsonpath='{.data.token}' | base64 --decode
```


<br/>

### 1-4) 내 PC에 인증서 설치

```sh
grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> client.crt
grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> client.key
openssl pkcs12 -export -clcerts -inkey client.key -in client.crt -out client.p12 -name "k8s-master-30"
```

kubecfg.p12 파일을 내 PC에서 인증서 등록

<br/>

### 1-5) Https 로 Dashboard 접근 후 Token 으로 로그인

```sh
https://192.168.0.30:6443/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
```

