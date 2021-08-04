
---
title: "Logging/Monitoring Architecture"
linkTitle: "Logging"
weight: 4
date: 2021-08-02
description: > 
  Promtail, Loki, Grafana
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/architecture/Logging.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>


<figure>
  <img src="/img/practice/architecture/Logging and Monitoring Architecture with Fluentd, ElasticSearch, Grafana for Kubernetes.jpg"
       alt="Logging and Monitoring Architecture with Fluentd, ElasticSearch, Grafana for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>


## 1. Basic Logging Construction

---


<figure>
  <img src="/img/practice/architecture/Node Level Logging for Kubernetes.jpg"
       alt="Node Level Logging for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 1-1) Deployment 생성

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-log
spec:
  selector:
    matchLabels:
      type: app
  template:
    metadata:
      labels:
        type: app
    spec:
      nodeSelector:
        kubernetes.io/hostname: k8s-node1
      containers:
      - name: container
        image: kubetm/app
```

api 호출

```sh
curl <pod-ip>:8080/hostname
curl <pod-ip>:8080/version
```

<br/>


### 1-2) Container Log 확인

kubectl exec로 Container 내부 로그파일 확인

```sh
kubectl exec <pod-name> -it -- /bin/sh
```

<br/>

kubectl logs로 Stdout 로그 확인

```sh
kubectl logs <pod-name>
kubectl logs <pod-name> --tail 10 --follow
```

<br/>

Docker Log Driver 설정 확인

```sh
cat /etc/docker/daemon.json
```

<br/>

Docker Container 로그 파일 확인

```sh
cd  /var/lib/docker/containers
docker ps
cd <container-id>
ls
```

<br/>


### 1-3) Node1에서 Log Link 확인
```sh
cd /var/log/pods
cd <Namespace>_<pod-name>_<pod-id>
cd container
ls -al

cd /var/log/containers
ls -al
```

<br/>


### 1-4) Event Log 확인

```sh
kubectl get events
kubectl get events | grep app-log
```

<br/>

### 1-5) Termination Log 확인

Termination API Call
```sh
curl <pod-ip>:8080/termination
```


Container Status 확인
```sh
kubectl describe pods <pod-name>
```

<br/>
<br/>



## 2. Loki Stack 설치

---

<figure>
  <img src="/img/practice/architecture/Loki Stack for Kubernetes.jpg"
       alt="Loki Stack for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>




### 2-1) Helm 설치

https://helm.sh/docs/intro/install/

```yaml
curl -O https://get.helm.sh/helm-v3.3.4-linux-amd64.tar.gz
tar -xf helm-v3.3.4-linux-amd64.tar.gz
cd linux-amd64
cp ./helm /usr/local/bin/
```

<br/>


### 2-2) loki-stack 설치

패키지 다운로드 
https://artifacthub.io/packages/helm/loki/loki-stack

```sh
helm repo add loki https://grafana.github.io/loki/charts
helm fetch loki/loki-stack --version 0.41.2

tar -xf loki-stack-0.41.2.tgz
cd loki-stack/
vi values.yaml
(Grafana Enable 설정)
```

helm 배포 / 삭제
```sh
kubectl create ns loki-stack
helm install loki-stack -f values.yaml . -n loki-stack
helm uninstall loki-stack -n loki-stack
```

<br/>

### 2-3) Grafana Service

```sh
kubectl edit -n loki-stack svc loki-stack-grafana
----
type: NodePort
nodePort : 30000
----

kubectl get secret --namespace loki-stack loki-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

<br/>
<br/>

## Referenece
---

### __Kubernetes__
  - __Logging Architecture__ : <https://kubernetes.io/docs/concepts/cluster-administration/logging/>


  