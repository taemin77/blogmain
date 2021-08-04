
---
title: "Helm 기초 다지기"
linkTitle: "Gettingstarted"
weight: 1
date: 2018-08-03
description: >
  Helm으로 Tomcat 배포
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/helm/basic/GettingStarted.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<br/><br/>

## 1. Installing Helm

---


Helm 설치 가이드 URL : https://helm.sh/ko/docs/intro/install/

<br/>

### 1-1) 릴리즈별 수동 다운로드


릴리즈별 다운로드 : https://github.com/helm/helm/releases/

```sh
curl -O https://get.helm.sh/helm-v3.4.2-linux-amd64.tar.gz
tar -zxvf helm-v3.4.2-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
```


### 1-2) 자동 최신버전 다운로드


```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### 1-3) 키워드 자동 완성 기능

```sh
source <(helm completion bash)

Linux: helm completion bash > /etc/bash_completion.d/helm 
MacOS: helm completion bash > /usr/local/etc/bash_completion.d/helm

[zsh] 
source <(helm completion zsh)
helm completion zsh > "${fpath[1]}/_helm"

[fish]
helm completion fish | source
helm completion fish > ~/.config/fish/completions/helm.fish
```


### 1-4) 버전 확인 명령어 

```sh
helm version
```


### 1-5) 쿠버네티스 config 파일 확인 

```sh
cd ~/.kube/
```

<br/>
<br/>




## 2. 차트 레포지토리 등록 및 톰켓 배포

---

### 2-1) 레포지토리 등록

Artifact Hub URL : https://artifacthub.io/

```sh
[등록]
helm repo add bitnami https://charts.bitnami.com/bitnami

[조회]
helm repo list

[Chart 찾기]
helm search repo bitnami | grep tomcat

[업데이트]
helm repo update

[삭제]
helm repo remove bitnami
```

### 2-2) Tomcat 배포

```sh
[Tomcat 배포]
helm install my-tomcat bitnami/tomcat --version 7.1.2 --set persistence.enabled=false

[NodePort 확인 및 접속]
kubectl get svc my-tomcat
http://<master-ip>:<nodePort>/
```

### 2-3) Tomcat 삭제

```sh
[배포 리스트 조회]
helm list

[배포 상태확인]
helm status my-tomcat

[Tomcat 삭제]
helm uninstall my-tomcat

[Pod 확인]
kubectl get pods
```


<br/>

### 2-4) 관리자 페이지에 접속이 되도록 다시 재설치


```sh

[Tomcat 배포]
helm install my-tomcat bitnami/tomcat --version 7.1.2 --set persistence.enabled=false,tomcatAllowRemoteManagement=1

[NodePort 확인 및 접속]
kubectl get svc my-tomcat
http://<master-ip>:<nodePort>/

[Tomcat 삭제]
helm uninstall my-tomcat
```


<br/>
<br/>





## 3. 톰켓 Chart 다운 및 배포

---

### 3-1) Chart 다운로드


```sh
[다운로드]
helm pull bitnami/tomcat --version 7.1.2

[압축풀기]
tar -xf ./tomcat-7.1.2.tgz

[Tomcat 배포]
helm install my-tomcat . -f values.yaml

[NodePort 확인 및 접속]
kubectl get svc my-tomcat
http://<master-ip>:<nodePort>/
```


<br/>
<br/>


## Referenece
---

### __Helm__
  - __Helm 홈페이지__ : <https://helm.sh>
  - __Helm 설치__ : <https://helm.sh/docs/intro/install/>

  