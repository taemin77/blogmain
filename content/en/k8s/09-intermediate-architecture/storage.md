
---
title: "Networking Architecture"
linkTitle: "Networking"
weight: 2
date: 2021-08-02
description: > 
  Pod Network(Pause Container, Network Plugin), Service Network(iptables, IPVS)
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/architecture/Storage.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>


<figure>
  <img src="/img/practice/architecture/Networking Architecture with PodNetwork, ServiceNetwork for Kubernetes.jpg"
       alt="Networking Architecture with PodNetwork, ServiceNetwork for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>




## 1. Pod Network - Pause Container
---

<figure>
  <img src="/img/practice/architecture/Pause Container on Pod Network for Kubernetes.jpg"
       alt="Pause Container on Pod Network for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

### 1-1) Pause Container


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-pause
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container1
    image: kubetm/p8000
    ports:
    - containerPort: 8000
  - name: container2
    image: kubetm/p8080
    ports:
    - containerPort: 8080
```

Pause Container 확인
```sh
docker ps | grep pod-pause
```

Pause Container 인터페이스 확인
```sh
docker ps | grep pod-pause
docker inspect <container-id> -f "{{json .NetworkSettings}}"
sudo ln -s /var/run/docker/netns /var/run/netns
ip netns exec <SandboxKey> ip a
```
<br/>

### 1-2) Calico Interface 확인

route 명령 설치
```sh
yum -y install net-tools
```

route으로 Pod IP와 연결 되어 있는 인터페이스 확인
```sh
route | grep cal
```

route로 확인된 가상인터페이스 ID가 호스트 네트워크에 있는지 확인
```sh
ip addr
```


<br/>

### 1-3) Pause Container Network Namespaces 확인


Pause Container와 타 Container간에 연결 확인
```sh
docker inspect <container-id> -f "{{json .HostConfig.NetworkMode}}"
```

Docker Container NetworkMode
```
NetworkMode - Sets the networking mode for the container. 
Supported standard values are: bridge, host, none, and container:<name|id>. 
Any other value is taken as a custom network’s name to which this container should connect to.
```

<br/>



## 2. Pod Network - Calico
---

<figure>
  <img src="/img/practice/architecture/Calico Network Plugin on Pod Network for Kubernetes.jpg"
       alt="Calico Network Plugin on Pod Network for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 2-1) Pod (source)


```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-src
  labels:
    type: src  
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node2
  containers:
  - name: container
    image: kubetm/init
    ports:
    - containerPort: 8080
```

<br/>

### 2-2) Pod (destination)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-dest
  labels:
    type: dest
spec:
  nodeSelector:
    kubernetes.io/hostname: k8s-node1
  containers:
  - name: container
    image: kubetm/app
    ports:
    - containerPort: 80
```

<br/>


### 2-3) Overlay Network(IP-in-IP) 트래픽 확인

Calico Overlay Network 확인
```sh
kubectl describe IPPool
```


Cluster의 Pod Network CIDR 확인
```sh
kubectl cluster-info dump | grep -m 1 cluster-cidr
```


<br/>


### 2-4) 트래픽 확인

tcpdump 설치
```sh
yum -y install tcpdump
```


트래픽 확인
```sh
route | grep cal
tcpdump -i <interface-name>
```

<br/>
<br/>


## 3. Service Network[clusterIP] - Calico
---


<figure>
  <img src="/img/practice/architecture/ClusterIP with Calico on Service Network for Kubernetes.jpg"
       alt="ClusterIP with Calico on Service Network for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 3-1) service (clusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-clusterip
spec:
  selector:
    type: dest
  ports:
  - port: 8080
    targetPort: 8080
  type: ClusterIP
```


<br/>

### 3-2) 트래픽 확인


```sh
route | grep cal
tcpdump -i <interface-name>
```

<br/>
<br/>



## 4. Service Network[NodePort] - Calico
---

<figure>
  <img src="/img/practice/architecture/NodePort with Calico on Service Network for Kubernetes.jpg"
       alt="NodePort with Calico on Service Network for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 4-1) service (NodePort)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-nodeport
spec:
  selector:
    type: dest
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 31080
  type: NodePort
```

<br/>

### 4-2) nodeport 확인


```sh
netstat -anp | grep 31080
```


<br/>

### 4-3) 트래픽 확인


```sh
tcpdump -i <interface-name>
```


<br/>
<br/>

## Referenece
---

### __Kubernetes__
  - __Service__ : <https://kubernetes.io/docs/concepts/services-networking/service/>

  
### __Others__
  - __The Almighty Pause Container__ : <https://www.ianlewis.org/en/almighty-pause-container>
  - __Calico__ : <https://docs.projectcalico.org/getting-started/kubernetes/>
  - __Comparing kube-proxy modes: iptables or IPVS?__ : <https://www.tigera.io/blog/comparing-kube-proxy-modes-iptables-or-ipvs/>
  
  