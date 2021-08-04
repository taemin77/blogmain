
---
title: "DaemonSet, Job, CronJob"
linkTitle: "DaemonSet"
weight: 12
date: 2018-07-3
description: > 
  
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/beginner/DaemonSet_CronJob_Job.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<figure>
  <img src="/img/practice/beginner/Controller with DatemonSet, Job, CronJob for Kubernetes.jpg"
       alt="Controller with DatemonSet, Job, CronJob for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>


## 1. DaemonSet
 
---
 
<figure>
  <img src="/img/practice/beginner/HostPort, NodeSelector with DaemonSet for Kubernetes.jpg"
       alt="HostPort, NodeSelector with DaemonSet for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


 
### 1-1) DaemonSet - HostPort
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-1
spec:
  selector:
    matchLabels:
      type: app
  template:
    metadata:
      labels:
        type: app
    spec:
      containers:
      - name: container
        image: kubetm/app
        ports:
        - containerPort: 8080
          hostPort: 18080
```

Command
```sh
curl 192.168.0.31:18080/hostname
```
<br/>
 
### 1-2) DaemonSet - NodeSelector
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: daemonset-2
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
        os: centos
      containers:
      - name: container
        image: kubetm/app
        ports:
        - containerPort: 8080
```


Label Add

```sh
kubectl label nodes k8s-node1 os=centos
kubectl label nodes k8s-node2 os=ubuntu
```
Label Remove

```sh
kubectl label nodes k8s-node2 os-
```

<br/>
<br/>



## 2. Job
---

 
### 2-1) Job1
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-1
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: container
        image: kubetm/init
        command: ["sh", "-c", "echo 'job start';sleep 20; echo 'job end'"]
      terminationGracePeriodSeconds: 0
```

<br/>
<br/>

<figure>
  <img src="/img/practice/beginner/Parrallelism, Completions with Job for Kubernetes.jpg"
       alt="Parrallelism, Completions with Job for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 2-2) Job2
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: job-2
spec:
  completions: 6
  parallelism: 2
  activeDeadlineSeconds: 30
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: container
        image: kubetm/init
        command: ["sh", "-c", "echo 'job start';sleep 20; echo 'job end'"]
      terminationGracePeriodSeconds: 0
```


<br/>
<br/>



## 3. CronJob

---

<figure>
  <img src="/img/practice/beginner/Allow with CronJob for Kubernetes.jpg"
       alt="Allow with CronJob for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### 3-1) CronJob
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cron-job
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: container
            image: kubetm/init
            command: ["sh", "-c", "echo 'job start';sleep 20; echo 'job end'"]
          terminationGracePeriodSeconds: 0
```


Manual 

```sh
kubectl create job --from=cronjob/cron-job cron-job-manual-001
```
Suspend

```sh
kubectl patch cronjobs cron-job -p '{"spec" : {"suspend" : false }}'
```
<br/>

{{% alert title="1.19버전 변경사항" color="warning" %}}
Cronjob 삭제시 Manual로 만든 Job도 같이 삭제됨
{{% /alert %}}


<br/>
<br/>

### 3-2) CronJob - ConcurrencyPolicy

<figure>
  <img src="/img/practice/beginner/ConcurencyPolicy with CronJob for Kubernetes.jpg"
       alt="ConcurencyPolicy with CronJob for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


### CronJob
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: cron-job-2
spec:
  schedule: "20,21,22 * * * *"
  concurrencyPolicy: Replace
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: container
            image: kubetm/init
            command: ["sh", "-c", "echo 'job start';sleep 140; echo 'job end'"]
          terminationGracePeriodSeconds: 0
```

<br/>



{{% alert title="1.19버전 변경사항" color="warning" %}}
__Replace 모드__ : 2min이 되었을 시 기존 Job은 삭제되고 (기존 Pod도 같이 삭제됨), 새 Job이(새 pod 생성) 만들어집니다.
{{% /alert %}}


<figure>
  <img src="/img/practice/beginner/ConcurencyPolicy 1.19 with CronJob for Kubernetes.jpg"
       alt="ConcurencyPolicy 1.19 with CronJob for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>


## Referenece
---

### __Kubernetes__
  - __Jobs__ : <https://kubernetes.io/docs/concepts/workloads/controllers/job/>
  - __CronJob__ : <https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/>
  - __Running Automated Tasks with a CronJob__ : <https://kubernetes.io/docs/tasks/job/automated-tasks-with-cron-jobs/>

