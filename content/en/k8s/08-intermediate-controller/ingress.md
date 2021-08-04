
---
title: "Ingress"
linkTitle: "Ingress"
weight: 2
date: 2021-08-02
description: > 
  Service Loadbalancing, Canary Upgrade
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/intermediate/Ingress.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/intermediate/Ingress with Service Loadbalancing and Canary for Kubernetes.jpg"
       alt="Ingress with Service Loadbalancing and Canary for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>

<figure>
  <img src="/img/practice/intermediate/Ingress with Loadbalancing and Canary Concept for Kubernetes.jpg"
       alt="Ingress with Loadbalancing and Canary Concept for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


## 1. Nginx Controller 
---

<figure>
  <img src="/img/practice/intermediate/Ingress with Install Nginx Controller for Kubernetes.jpg"
       alt="Ingress with Install Nginx Controller for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 1-1) Nginx 설치
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.1/deploy/static/mandatory.yaml
```

### 1-2) NodePort Service 생성
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.27.1/deploy/static/provider/baremetal/service-nodeport.yaml
```


<br/>


## 2. Service Loadbalancing
---

<figure>
  <img src="/img/practice/intermediate/Ingress with Service Loadbalancing for Kubernetes.jpg"
       alt="Ingress with Service Loadbalancing for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 2-1) Shopping Page
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-shopping
  labels:
    category: shopping
spec:
  containers:
  - name: container
    image: kubetm/shopping
---
apiVersion: v1
kind: Service
metadata:
  name: svc-shopping
spec:
  selector:
    category: shopping
  ports:
  - port: 8080
```



### 2-2) Customer Center
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-customer
  labels:
    category: customer
spec:
  containers:
  - name: container
    image: kubetm/customer
---
apiVersion: v1
kind: Service
metadata:
  name: svc-customer
spec:
  selector:
    category: customer
  ports:
  - port: 8080
```


### 2-3) Order Service
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-order
  labels:
    category: order
spec:
  containers:
  - name: container
    image: kubetm/order
---
apiVersion: v1
kind: Service
metadata:
  name: svc-order
spec:
  selector:
    category: order
  ports:
  - port: 8080
```


### 2-4) Ingress
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: service-loadbalancing
spec:
  rules:
  - http:
      paths:
        - path: /
          backend:
            serviceName: svc-shopping
            servicePort: 8080
        - path: /customer
          backend:
            serviceName: svc-customer
            servicePort: 8080
        - path: /order
          backend:
            serviceName: svc-order
            servicePort: 8080
```


```sh
curl 192.168.0.30:30431/
curl 192.168.0.30:30431/order
curl 192.168.0.30:30431/customer
```





<br/>
<br/>



## 3. Canary Upgrade
---

<figure>
  <img src="/img/practice/intermediate/Ingress with Canary Upgrade for Kubernetes.jpg"
       alt="Ingress with Canary Upgrade for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

### 3-1) App V1
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-v1
  labels:
    app: v1
spec:
  containers:
  - name: container
    image: kubetm/app:v1
---
apiVersion: v1
kind: Service
metadata:
  name: svc-v1
spec:
  selector:
    app: v1
  ports:
  - port: 8080
```


### 3-2) App V2
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-v2
  labels:
    app: v2
spec:
  containers:
  - name: container
    image: kubetm/app:v2
---
apiVersion: v1
kind: Service
metadata:
  name: svc-v2
spec:
  selector:
    app: v2
  ports:
  - port: 8080
```



### 3-3) Ingress - default
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: app
spec:
  rules:
  - host: www.app.com
    http:
      paths:
      - backend:
          serviceName: svc-v1
          servicePort: 8080
```


```sh
# Centos HostName 등록
cat << EOF >> /etc/hosts
192.168.0.30 www.app.com
EOF

curl www.app.com:30431/version
```

### 3-4) Ingress - weight
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: canary-v2
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
spec:
  rules:
  - host: www.app.com
    http:
      paths:
      - backend:
          serviceName: svc-v2
          servicePort: 8080
```

```sh
while true; do curl www.app.com:30431/version; sleep 1; done
```

### 3-5) Ingress - header
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: canary-kr
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-by-header: "Accept-Language"
    nginx.ingress.kubernetes.io/canary-by-header-value: "kr"
spec:
  rules:
  - host: www.app.com
    http:
      paths:
      - backend:
          serviceName: svc-v2
          servicePort: 8080
```

```sh
curl -H "Accept-Language: kr" www.app.com:30431/version
```


<br/>
<br/>



## 4. SSL
---

<figure>
  <img src="/img/practice/intermediate/Ingress with SSL for Kubernetes.jpg"
       alt="Ingress with SSL for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>




### 4-1) App V1
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-https
  labels:
    app: https
spec:
  containers:
  - name: container
    image: kubetm/app
---
apiVersion: v1
kind: Service
metadata:
  name: svc-https
spec:
  selector:
    app: https
  ports:
  - port: 8080
```


### 4-2) Ingress - ssl
```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: https
spec:
  tls:
  - hosts:
    - www.https.com
    secretName: secret-https
  rules:
    - host: www.https.com
      http:
        paths:
        - backend:
            serviceName: svc-https
            servicePort: 8080
```

### 4-3) Secret

```sh
# 인증서 생성
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=www.https.com/O=www.https.com"

# Secret 생성
kubectl create secret tls secret-https --key tls.key --cert tls.crt

# Windows HostName 등록
파일 위치 : C:\Windows\System32\drivers\etc\hosts
192.168.0.30 www.https.com
```

<br/>
<br/>

## Referenece
---
### __Kubernetes__
  - __Ingress__ : <https://kubernetes.io/docs/concepts/services-networking/ingress/>
  - __Ingress Controllers__ : <https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/>
  
<br/>

### __Others__
  - __Nginx Installation Guide__ : <https://kubernetes.github.io/ingress-nginx/deploy/#prerequisite-generic-deployment-command>