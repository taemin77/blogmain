
---
#title: "Case 3. 구글 클라우드 플랫폼"
#linkTitle: "Case 3"
weight: 5
date: 2021-08-02
#description: > 
 # 해당 가이드는 강의에서 더이상 지원하지 않습니다.
---

<figure>
  <img src="/img/practice/appendix/Google Cloud Platform Installation for Kubernetes.jpg"
       alt="Google Cloud Platform Installation for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



<br/>
<br/>


<figure>
  <img src="/img/practice/appendix/Join Google Cloud Platform for Kubernetes.jpg"
       alt="Join Google Cloud Platform for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


## 1-1) Join GCP
---




### 1-1-1) Join

아래 사이트에 들어가서 상단에 [무료로 시작하기] 버튼 클릭
<br/>
>https://cloud.google.com/

<br/>

```sh
1) Country : South Korea
2) Terms of Service : 체크 후 [CONTINUE]
3) 이름 및 주소 확인 후 Payment method 입력
```

자신의 신용카드 번호를 입력하지만 크레딧을 다 소모하거나 12개월이 지나도 자동으로 결재가 되진 않습니다.




<br/>
<br/>

<figure>
  <img src="/img/practice/appendix/Create Master and Node Cluster for Kuebernetes.jpg"
       alt="Create Master and Node Cluster for Kuebernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


## 2-1) Create Cluster
---





### 2-1-1) Cluster 생성

Kubernetes Engine > Clusters > [CREATE CLUSTER] 클릭

```sh
1. Name(이름) : k8s-cluster
2. Location Type(위치 유형) : Zonal (영역)
3. Zone(영역) : asia-east1-a 부터 asia-east2-c 중 하나 선택
4. Specify default node locations (기본 노드 위치 지정) : 체크 안함
5. Master version (마스터 버전) : 정적버전 (Static version)
6. Static version (정적버전) : 1.15.11-gke.15
7. 맨 하단에 create (만들기) 클릭
```

node들은 자동으로 3개가 만들어집니다.


<br/>
<br/>


<figure>
  <img src="/img/practice/appendix/Connect Google Cloud Platform for Kubernetes.jpg"
       alt="Connect Google Cloud Platform for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



## 3-1) Connect GCP
---

 
 
### 3-1-1) Install SDK 
아래 URL에서 Windows용 GCP SDK 설치
<br/>

>https://cloud.google.com/sdk/docs/quickstart-windows

<br/>

다른 운영체제에서는 아래 내용 참조
<br/>

>https://cloud.google.com/sdk

```sh
1. [Google Cloud SDK 설치 프로그램] 다운로드
2. Next > Next 를 통해 설치 후 마지막에 4가지 체크항목 모두 선택 후 [Finish]
```
<br/>


## 3-2) Setting SDK Shell

### 3-2-1) Install SDK 
바탕화면에 설치된 [Google Cloud SDK Shell] 실행하면 아래 내용을 물어봅니다.

```sh
1. Pick configuration to use : [1] Re-initialize
2. Choose the account : [1] GCP 등록 구글 계정 선택, 없으면 [2] Log in with a new account
   [2] 선택시  나오는 URL 복사해서 웹 브라우저에 복사 후 인증 필요
3. Pick cloud project to use : GCP [My First Project] ID 선택 
   (GCP 브라우저에서 확인필요 [강의 영상 참조])
4. Do you want to configure a default Compute Region and Zone : Y
5. [46] asia-east2-c 선택  
   (2-1-1) Cluster 생성시 [3. Zone(영역]에 선택한 내용과 동일하게 선택)
```

<br/>


### 3-2-2) gcloud 업그레이드 및 kubectl 설치

```sh
gcloud components update
```

별도로 팝업창이 안뜰 수 있음 All components are up to date 문구 확인


```sh
gcloud components install kubectl
```

<br/>
GCP > Kubernetes Engine > Clusters > k8s-cluster 리스트에 [Connect] 버튼 클릭하여 나오는 팝업에서 아래 내용 복사
<br/>

`gcloud container clusters get-credentials k8s-cluster --zone asia-east2-c --project turnkey-conduit-258023`

<br/>

Local GCP Shell에 붙여 놓기를 한 후 아래 Node 조회 명령어로 

```sh
kubectl get nodes
```


<br/>
<br/>


<figure>
  <img src="/img/practice/appendix/Add Plugin Dashboard for Kubernetes.jpg"
       alt="Add Plugin Dashboard for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

## 4-1) Dashboard
---



### 4-1-1) Dashboard 설치 

해당 설정은 교육목적으로 권한 설정을 모두 해제하는 방법이기 때문에 프로젝트에서 사용하실때는 이점 유의바래요
<br/>
>https://github.com/kubernetes/dashboard


```sh
kubectl apply -f https://raw.githubusercontent.com/kubetm/kubetm.github.io/master/sample/practice/appendix/gcp-kubernetes-dashboard.yaml
```

### 4-1-2) proxy 띄우기	

```sh
kubectl proxy
```

### 4-1-3) 접속 URL 

```sh
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```


