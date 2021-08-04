
---
title: "CKA 자격증 개요"
linkTitle: "concept"
weight: 2
date: 2021-08-02
description: > 
  시험 관련 전반적인 내용 설명
---



<figure>
  <img src="/img/practice/cka/Certified Kubernetes Administrator for Kubernetes.jpg"
       alt="Certified Kubernetes Administrator for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>



## 시험 접수 및 기본 정보

---
### __시험비__ : 300불   `재시험 기회 1번 포함`
  - __접수__ : https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/
  - 할인 쿠폰 : 
```
구글 검색에 [cka exam coupon code] 키워드로 검색하면 최소 30%는 찾을 수 있음
ex: KUBERNETES30, ANYWHERE30
```

<br/>

### __버전__ : 꼭 버전 확인! 해당 내용은 1.17 기준

<br/>

### __시험시간__ : 3시간   

<br/>

### __시험문항__ : 24문항   
`난이도에 따라 1점 ~ 9점, 모두 실습 형태`

<br/>

### __합격기준__ : 74점 이상   
`시험 완료 후 36시간 후에 이메일로 전송`

<br/>


### __시험 관련 My Portal URL__

  - __시험 환경__ : 인터넷 연결, Chrome 및 Chromium 브라우저 사용, 웹캠, 마이크, 신분증
  - __URL__ : https://www.examslocal.com/ScheduleExam/Home/CompatibilityCheck 
```sh
1. Option3에서 Linux Foundation 선택
2. Certified Kubernetes Administrator (CKA) - English 선택 
3. [Go] 클릭
4. Operating System, Web Browser, Webcam/Microphone, Google Chrome Extension, Ports, Bandwidth의 Status 체크 확인
 - Google Chrome Extension : Innovative Exams Screensharing 크롬 웹 스토어를 통해 설치
```

<br/>

  - __시험 날짜__ : 제시되는 스케줄 내에서 본인의 일정에 따라 정할 수 있음

  - __시험가이드__ : <https://training.linuxfoundation.org/cncf-certification-candidate-resources>
 
<br/>

<br/>



## 시험 준비

---

### __시험 범위__ 
  - https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka
  
<br/>

### __오픈북 시험__ : 
  - __탭 하나만 이용해서 아래 Site만 접속 가능__ 
<br/>
-- https://kubernetes.io/docs/
<br/>
-- https://github.com/kubernetes/
<br/>
-- https://kubernetes.io/blog/
  - __해당 URL의 하위 도메인들은 모두 가능__ 
  - __해당 사이트들내에서 클릭시 외부로 연결되어 있는 사이트는 사용 불가__ 


<br/>

### __시험준비 관련 URL__
  - __시험 범위 별 docs 링크1__ : https://github.com/leandrocostam/kubernetes-certified-administrator-prep-guide
  - __시험 범위 별 docs 링크2__ : https://github.com/walidshaari/Kubernetes-Certified-Administrator

<br/>

### __시험준비 주의사항__
  - 제가 CKA편을 통해서 드리는 설명은 CKA를 통해서 더 쿠버네티스를 잘 알고자 함이 아닙니다.
  - 그냥 최소한의 시간으로 시험에 합격하는 방법을 말씀 드릴려고 하기 때문에, kubernetes-the-hard-way나 외국 Training 강의를 추천하지는 않을께요.
  - 기본적으로 제 강의를 다 보시고 내용을 이해한 수준에서 해당 [CKA편 시험공부] 강의만 들어도 80점 이상은 받을 수 있다고 생각합니다.
  - 하지만 제가 CKA시험을 지속적으로 관찰하고 있지는 않을 것이기 때문에 향후 시험 내용이 크게 변경된다면 그런 부분에 대해서는 보장을 드릴 수는 없습니다.
  - 그리고 만약 시험 내용이 많이 변경됐다는 걸 알게 되다면 해당 강의는 내릴 예정이고요.
  - 저는 문제를 모두 풀었지만 만점을 받은 것은 아니여서 어느정도 오답이 있을 수 있다는 점도 고려 바랍니다.
  - 시험일 : 2020-04-10, 시험버전 : 1.17, 합격점수 : 89점
  - 이후 시험을 보신 분들 중에서 저에게 변경사항이 이나, 강의 내용중 잘못된 부분을 알려주신다면 지속적으로 업데이트 하겠습니다.
  
  
<br/><br/>

## 시험 시작

---

### __시험전 준비__
  - 시험 작업 환경 세팅 : 화면 및 웹캠/마이크 공유, 신분증 검사, 웹켐으로 방 주변을 보여줌. 
  <br/>
(개인적으로 화면 공유버튼을 눌러도 공유에 녹색불이 안들어와서 당황스러웠는데, 브라우저 캐시를 지우고 다시 접속해보라고 하고, 컴퓨터도 껏다가 다시 접속해 보라고함, 컴퓨터를 껐다킨 후 재접속하니까 정상적으로 작동됨)
  - 컴퓨터 운영체제가 뭔지 물어보고, 다른 프로그램들이 실행중인지 `Task Manager`도 실행해 보라고함.
  - 끝으로 시험중 주의사항에 대해서 간단한 안내사항들을 공지해주고 시험 시작 모든 대화는 채팅으로 진행됨
  - 시험 안내서에 시험동안 열어놔야 하는 Port들이 있었는데, 방화벽을 모두 꺼놓고 시험을 봄
  
  
### __시험중__
  - __레퍼런스 사이트__ : 크롬 브라우저를 하나더 열어놓고, 탭 하나만 사용, 사전에 필요한 Url들은 `bookmark`로 정리해 놓으면 좋음
  - __메모장__ : 시험 툴에서 제공되는 메모장이 있음. 레퍼런스 사이트의 내용을 복붙해서 수정할때 주로 사용했고, 메모장에 있는 내용을 복사할때 드레그가 잘 안됨
  - __복붙__ : 레버런스 사이트나 메모장에는 Ctrl+C, V가 잘되나 작업 Console에는 해당 명령이 안먹힘, 가이드 문서에 대체 명령이 있으나, 마우스 우클릭 하면 나오는 메뉴에 붙여놓기 기능이 있어서 그것을 사용했음
  - __남은시간__ : 왼쪽 상단쯤에 timer가 있는데 남은 시간이 표시되는게 아니라 [======] 이렇게 바 형태로 남은 시간의 양을 알려줌, 그리고 채팅으로 시간 물어보면 정확히 남은 시간을 알려줌 
  - __주의행동__ : 문제 풀다가 무의식적으로 입을 가리거나 문제를 입으로 읽었더니 바로 채팅창으로 주의를 줌. 자세한 주의행동에 대해서는 가이드 문서에 행동규칙 참고 
  - __작업Cluster__ : 기본적으로 Host(node-1)에서 [kubectl config user-context k8s] 명령을 통해서 원하는 Cluster로 작업 명령을 날림. 시험 문제마다 어떤 클러스터에 연결해서 작업을 하라는 내용이 나와있고, 70%정도가 k8s Cluster에서 작업함. (ik8s 용도는 쿠버네티스 master, node 설치), ssh로 연결시 user권한이고, root 권한이 아니기 때문에 sudo -i 명령을 써서 작업하는 경우가 있는데 작업 후 node-1까지 돌아갈려면 exit를 두번 해야함. (실수로 한번만 해서 worker를 node-1으로 착각할 수 있음) 

<figure>
  <img src="/img/practice/cka/Cka Clusters.jpg"
       alt="Cka Clusters."
       class="mt-3 mb-3 border border-info rounded" />
</figure>
  - __콘솔__ : 단일 콘솔만 사용가능, 멀티 플렉서 Screen인 tmux을 권장하는데 현재 사용하고 있으면 쓰면 좋을 수 있으나, 아니면 굳이 tmux 사용법을 배워서 써야될만큼 멀티 콘솔로 작업을 해야할 필요성은 못 느꼈음.
  
  