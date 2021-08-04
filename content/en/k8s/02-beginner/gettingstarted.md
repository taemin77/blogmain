
---
title: "Getting-Started Kubernetes!"
linkTitle: "Gettingstarted"
weight: 1
date: 2021-08-02
description: >
  
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/beginner/Getting_Started3.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>


<figure>
  <img src="/img/practice/beginner/Getting started Kubernetes.jpg"
       alt="Getting started Kubernetes"
       class="mt-3 mb-3 border border-info rounded" />
</figure>


{{% alert title="실습 참고사항" color="warning" %}}
이 실습 강의의 목적은 일반 서버와 도커, 그리고 쿠버네티스 환경의 차이점에 대해 대략적인 흐름을 이해하기 위함입니다.
그렇기 때문에 강의를 위한 별도의 사전 구축 내용들은 설명되지 않았고, 이후 쿠버네티스를 설치하고 [기초편]부터 실습을 진행하시면 되세요.
하지만 기존에 도커를 잘 아시는 분께서는 자신의 환경에 마춰 실습해 보셔도 무관합니다.
{{% /alert %}}



<br/>
<br/>



## 1. Linux
---

<figure>
  <img src="/img/practice/beginner/Getting started Hello World on Linux.jpg"
       alt="Getting started Hello World on Linux."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 1-1) hello.js
```javascript
var http = require('http');
var content = function(req, resp) {
 resp.end("Hello Kubernetes!" + "\n");
 resp.writeHead(200);
}
var w = http.createServer(content);
w.listen(8000);
```
```sh
node hello.js
```


<br/>
<br/>


## 2. Docker 
---


<figure>
  <img src="/img/practice/beginner/Getting started Hello World on Docker.jpg"
       alt="Getting started Hello World on Linux."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 2-1) Dockerfile
```sh
FROM node:slim
EXPOSE 8000
COPY hello.js .
CMD node hello.js
```

### 2-2) Docker Hub Site
https://hub.docker.com/

### 2-3) Docker Container Run
```sh
docker build -t kubetm/hello .
-t : 레파지토리/이미지명:버전

docker images
docker run -d -p 8100:8000 kubetm/hello
-d : 백그라운드 모드
-p : 포트변경

docker ps
docker exec -it c403442e8a59 /bin/bash
```


### 2-4) Docker Image Push

```sh
docker login
docker push kubetm/hello
```


<br/>
<br/>



## 3. Kubernetes 
---

<figure>
  <img src="/img/practice/beginner/Getting started Hello World on Kubernetes.jpg"
       alt="Getting started Hello World on Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 3-1) Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-pod
  labels:
    app: hello
spec:
  containers:
  - name: hello-container
    image: kubetm/hello
    ports:
    - containerPort: 8000
```

### 3-2) Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-svc
spec:
  selector:
    app: hello
  ports:
    - port: 8200
      targetPort: 8000
  externalIPs:
  - 192.168.0.30
```
