
---
title: "Case 2. OS(windows) + Network(Bridge Mode) + 가상화(VirtualBox-CentOS) + k8s(v1.15.5)"
linkTitle: "설치 Case 2"
weight: 2
date: 2021-08-02
description: > 
  윈도우나 맥이 설치되어 있는 Desktop가 1대 있고, 그위에 VirtualBox와 같은 도구를 이용해서 VM을 만드는 경우
---


<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/beginner/Kubernetes_Install2.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/appendix/VirtualBox Installation for Kubernetes.jpg"
       alt="VirtualBox Installation for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



<br/>
<br/>


<figure>
  <img src="/img/practice/appendix/Setting Physical Server for Kubernetes.jpg"
       alt="Setting Physical Server for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


## 1-1) Install Virtualbox
---


### 1-1-1) virtualbox 다운로드 및 설치

[윈도우10 버전으로 진행] 아래 경로에서 [Windows hosts] 클릭 하여 다운로드 후 설치 (별다른 변경없이 Next만 함)
<br/>
>https://www.virtualbox.org/wiki/Downloads

<br/>

Mac 사용자 참고 URL
<br/>
>https://www.virtualbox.org/wiki/Mac%20OS%20X%20build%20instructions

<br/>

### 1-1-2) CentOS Download
   
아래 경로로 들어가서 원하는 경로에서 Minimal 버전의 파일 다운로드
<br/>
>http://isoredirect.centos.org/centos/7/isos/x86_64/

<br/>

### 1-1-3) 원격접속 툴 MobaXterm 설치

꼭 MobaXterm이 아닌 각자 편한 원격접속 툴을 사용하셔도 되세요.
<br/>
아래 예제는 MobaXterm를 설치하고 필요한 Host 등록 예제입니다.
<br/>
>https://mobaxterm.mobatek.net/

```sh
- [GET MOBAXTERM NOW] 버튼 클릭
- Free 버전 [Download now]
- Installer editon 다운로드 및 실행
- Sessions > SSH > Remote host : 192.168.0.30 > [Bookmark settings] Session name : master-192.168.0.30 > [ok]
- Sessions > SSH > Remote host : 192.168.0.31 > [Bookmark settings] Session name : node1-192.168.0.31 > [ok]
- Sessions > SSH > Remote host : 192.168.0.32 > [Bookmark settings] Session name : node2-192.168.0.32 > [ok]
```

<br/>
<br/>


<figure>
  <img src="/img/practice/appendix/Create VM for Kubernetes.jpg"
       alt="Create VM for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



## 2-1) Setting VM
---

### 2-1-1) VM 스펙 설정
메모리 및 디스크는 각자 상황에 맞게 참고해서 VM 설정

```sh
1. [VM 생성 1단계] 머신 > 새로 만들기 클릭
2. [VM 생성 1단계] 이름 : k8s-master, 종류: Linux, 버전: Other Linux(64-bit)
3. [VM 생성 2단계] 메모리 : 4096 MB
4. [VM 생성 3단계] 하드디스크 : 지금 새 가상 하드 디스크 만들기 (VDI:VirtualBox 디크스 이미지, 동적할당, 150GB)
5. [VM 생성 후 시스템 설정] 프로세서 개수 : CPU 2개
6. [VM 생성 후 저장소 설정] 컨트롤러:IDE 하위에 있는 광학드라이브 클릭 > CentOS 이미지 선택 후 확인
7. [VM 생성 후 네트워크 설정] VM 선택 후 설정 버튼 클릭 > 네트워크 > 어댑터 1 탭 > 다음에 연결됨 [어댑터에 브리지] 선택
   - IP를 할당 받을 수 없는 경우  [NAT 네트워크] 
8. 시작
```

<br/>
<br/>


## 2-2) Install CentOS
---

### 2-2-1) CentOS 설치

4번 단계에서 `8.8.8.8`는 Google DNS입니다. 원하는 DNS 쓰셔도 되요.

```sh
1. Test this media & install CentOS 7
2. Language : 한국어 
3. Disk 설정 [시스템 > 설치 대상]
   - [기타 저장소 옵션 > 파티션 설정] 파티션을 설정합니다. [체크] 후 [완료]
   - 새로운 CentOS 설치 > 여기를 클릭하여 자동으로 생성합니다. [클릭]
   - /home [클릭] 후 용량 5.12 GiB로 변경 [설정 업데이트 클릭]
   - / [클릭] 후 140 GiB 변경 후 [설정 업데이트 클릭]
   - [완료], [변경 사항 적용]
4. 네트워크 설정 [시스템 > 네트워크 및 호스트명 설정]
   - 호스트 이름: k8s-master [적용]
   - 이더넷 [설정]
      [일반] 탭
      - 사용 가능하면 자동으로 이 네트워크에 연결 [체크]
      [IPv4 설정] 탭
      - 방식: 수동으로 선택, 
      - [Add] -> 주소: 192.168.0.30, 넷마스크 : 255.255.255.0, 게이트웨이: 192.168.0.1, DNS 서버 : 8.8.8.8
      - [저장][완료]      
 5. 설치시작
6. [설정 > 사용자 설정] ROOT 암호 설정 
7. 설치 완료 후 [재부팅]
```

<br/>
<br/>

<figure>
  <img src="/img/practice/appendix/Docker, Kubernetes Installation.jpg"
       alt="Docker, Kubernetes Installation."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


## 3-1) Pre-Setting
---

### 3-1-1) SELinux 설정


Ubuntu나 Debian등 다른 OS를 설치하시는 분들께서는 아래 경로에서 명령어 참고 바래요
<br/>
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
<br/>
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker
<br/>
쿠버네티스가 Pod Network에 필요한 호스트 파일 시스템에 액세스가 가능하도록 하기 위해서 필요한 설정이예요
<br/>
아래 설정으로 SELinux을 permissive로 변경해야하고 

```sh
setenforce 0
```
리부팅시 다시 원복되기 때문에 아래 명령을 통해서 영구적으로 변경 해야되요

```sh
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```

아래 명령어를 실행해서 `Current mode:permissive` 내용 확인

```sh
sestatus
```

<br/>

### 3-1-2) 방화벽 해제

firewalld 비활성화

```sh
systemctl stop firewalld && systemctl disable firewalld
```

NetworkManager 비활성화

```sh
systemctl stop NetworkManager && systemctl disable NetworkManager
```

### 3-1-3) Swap 비활성화
Swap 사용에 관련해서는 많은 의견이 있어요.
<br/>
>https://github.com/kubernetes/kubernetes/issues/53533
<br/>
위 내용을 참고하셔서 swap 사용시의 고려해야할 점을 확인하시고 일단 여기선 사용하지 않도록 설정할께요.

```sh
swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab
```

### 3-1-4) iptables 커널 옵션 활성화
RHEL이나 CentOS7 사용시 iptables가 무시되서 트래픽이 잘못 라우팅되는 문제가 발생한다고 하여 아래 설정이 추가되요

```sh
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 3-1-5) 쿠버네티스 YUM Repository 설정

YUM에 대해서 좀더 상세한 내용이 궁금한 분께서는 아래 싸이트가 잘 정리되어 있는거 같아 링크 첨부했어요.
<br/>
>https://www.lesstif.com/display/1STB/yum

```sh
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
```

### 3-1-6) Centos Update

```sh
yum update -y 
```

### 3-1-7) hosts 등록
계획된 master와 node의 호스트 이름과 IP를 모두 등록해주세요. 안하시면 추후 kubeadm init시 Host이름으로 IP를 찾을 수 없다고 에러가 나요.

```sh
cat << EOF >> /etc/hosts
192.168.0.30 k8s-master
192.168.0.31 k8s-node1
192.168.0.32 k8s-node2
EOF
```


<br/>
<br/>


## 3-2) Install 
---



### 3-2-1) Docker 설치 

도커 설치 전에 필요한 패키지 설치 

```sh
yum install -y yum-utils device-mapper-persistent-data lvm2 
```

 도커 설치를 위한 저장소 를 설정 

```sh
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

도커 패키지 설치 

```sh
yum update -y && yum install -y docker-ce-18.06.2.ce
```

```sh
mkdir /etc/docker
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

mkdir -p /etc/systemd/system/docker.service.d
```

### 3-2-2) Kubernetes 설치

```sh
yum install -y --disableexcludes=kubernetes kubeadm-1.15.5-0.x86_64 kubectl-1.15.5-0.x86_64 kubelet-1.15.5-0.x86_64
```






<br/>
<br/>


<figure>
  <img src="/img/practice/appendix/Clone VM on Node For Kubernetes.jpg"
       alt="Clone VM on Node For Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


## 4-1) Clone VM
---




### 4-1-1) 시스템 shutdown

여기까지 만든 이미지를 복사해 놓기 위해서 Master를 잠시 Shutdown 시켜요.

```sh
shutdown now
```

### 4-1-2) VM 복사

VirtualBox UI를 통해 Master 선택 후 마우스 우클릭을 해서 [복제] 버튼 클릭

```sh
1. 이름 : k8s-node1, MAC 주소정책 : 모든 네트워크 어댑터의 새 MAC 주소 생성
2. 복제방식 : 완전한 복제
```

node2도 반복


<br/>
<br/>

## 4-2) Config Node
---

   
### 4-2-1) Network 변경하기

VirtualBox UI에서 k8s-node1을 시작 시키면 뜨는 Console 창을 통해 아래 명령어 입력
<br/>
Host의 Ip Address를 변경하기 위해 아래 명령어로 설정을 열고

```sh
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```
`IPADDR=` 부분을 해당 Node의 IP (192.168.0.31)로 변경해주세요

```sh
...
DEVICE="etho0"
ONBOOT="yes"
IPADDR="192.168.0.31"
...
```

그리고 아래 명령어로 네트워크 재시작

```sh
systemctl restart network
```

### 4-2-2) Host Name 변경
해당 Node의 Host 이름을 변경해주세요

```sh
hostnamectl set-hostname k8s-node1
```

이와 같은 방식으로 k8s-node2(192.168.0.32) 도 설정합니다.




<br/>
<br/>


<figure>
  <img src="/img/practice/appendix/Initialize Master and Join Node for Kubernetes.jpg"
       alt="Initialize Master and Join Node for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


## 5-1) Master
---



### 5-1-1) 도커 및 쿠버네티스 실행
도커 실행

```sh
systemctl daemon-reload
```

```sh
systemctl enable --now docker
```

아래 명령어를 입력하면 image를 다운받는 내용이 나오면서 중간에  `Hello for Docker!` 가 보이면 설치 확인되면 설치가 잘 된거예요.

```sh
docker run hello-world
```

쿠버네티스 실행

```sh
systemctl enable --now kubelet
```


### 5-1-2) 쿠버네티스 초기화 명령 실행

kubeadm init 명령관련 해서 상세 내용이 궁금하신 분은 아래 싸이트 참고하세요.
>https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init/
<br/>
`pod-network-cidr` 를 설정하면 Pod의 IP가 자동으로 생성될때 해당 network으로 생성되요

<br/>
`service-cidr` 를 설정하면 Service의 IP가 자동으로 생성될때 해당 대역으로 생성되요 `Default: 10.96.0.0/12`

```sh
kubeadm init --pod-network-cidr=20.96.0.0/12 --apiserver-advertise-address=192.168.0.30
```

실행 후 `[Your Kubernetes master has initialized successfully!]` 문구를 확인하고 아래 내용 복사해서 별도로 저장해 둡니다. 
<br/>
kubeadm join 192.168.0.30:6443 --token ki4szr.t3wondaclij6d1a3 \
    --discovery-token-ca-cert-hash sha256:2370f0451342c6e4bd0d38f6c2511bda5c50374c85e9c09da28e12dd666d5987
    
### 5-1-3) 환경변수 설정
root 계정을 이용해서 kubectl을 실행하기 위한 환경 변수를 설정

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 5-1-4) kubectl 자동완성 기능 설치
kubectl 사용시 [tab] 버튼을 이용해서 다음에 올 명령어 리스트를 조회 할 수 있어요.
<br/>
명령 실행 후 바로 적용이 안되기 때문에 접속을 끊고 다시 연결 후에 사용 가능합니다. 

```sh
yum install bash-completion -y
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
```


## 5-2) Node
---



### 5-2-1) 도커 및 쿠버네티스 실행

도커 실행

```sh
systemctl daemon-reload
```

```sh
systemctl enable --now docker
```

쿠버네티스 실행

```sh
systemctl enable --now kubelet
```

### 5-2-2) Node 연결
Master Init 후 복사 했었던 내용 붙여넣기

```sh
kubeadm join 192.168.0.30:6443 --token 7xd747.bfouwf64kz437sqs \
    --discovery-token-ca-cert-hash sha256:ec75641cd258f2930a7f73abfe540bb484eb295ad4500ccdaa166208f97c5117
```

### 5-2-3) Node 연결 확인
Master 서버에 접속해서 아래 명령 입력 후 추가된 Node가 보이는지 확인 (Status는 NotReady)

```sh
kubectl get nodes
```




<br/>
<br/>

<figure>
  <img src="/img/practice/appendix/Add Networking and Dashboard Plugin for Kubernetes.jpg"
       alt="Add Networking and Dashboard Plugin for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

## 6-1) Networking
---




### 6-1-1) Calico 설치

Kubernetes Cluster Networking에는 많은 Plugin들이 있는데 그중 Calico 설치에 대한 내용 입니다.
<br/>
>https://docs.projectcalico.org/v3.9/getting-started/kubernetes/
<br/>
Calico는 기본 192.168.0.0/16 대역으로 설치가 되는데, 그럼  실제 VM이 사용하고 있는 대역대와 겹치기 때문에 수정을 해서 설치해야 할 경우

```sh
curl -O https://docs.projectcalico.org/v3.9/manifests/calico.yaml
sed s/192.168.0.0\\/16/20.96.0.0\\/12/g -i calico.yaml
kubectl apply -f calico.yaml
```

calico와 coredns 관련 Pod의 Status가 Running인지 확인 

```sh
kubectl get pods --all-namespaces
```




## 6-2) Dashboard
---




### 6-2-1) Dashboard 설치 

해당 설정은 교육목적으로 권한 설정을 모두 해제하는 방법이기 때문에 프로젝트에서 사용하실때는 이점 유의바래요
<br/>
>https://github.com/kubernetes/dashboard


```sh
kubectl apply -f https://kubetm.github.io/documents/appendix/kubetm-dashboard-v1.10.1.yaml
```


### 6-2-2) 백그라운드로 proxy 띄우기	
`--address`에 자신의 Host IP 입력 

```sh
nohup kubectl proxy --port=8001 --address=192.168.0.30 --accept-hosts='^*$' >/dev/null 2>&1 &
```

* 만약 Master 노드가 Restart 됐을 경우 이 부분은 다시 실행해 줘야 합니다.


### 6-2-3) 접속 URL 

```sh
http://192.168.0.30:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

### 6-2-4) 언어 설정변경

```sh
Chrome 설정 > 언어 > 언어(한국어) > 원하는 언어를 위로 추가
```