
---
title: "내 차트 만들기1"
linkTitle: "Step1"
weight: 1
date: 2021-08-02
description: >
  내장 객체, 변수 주입 우선순위, 사용자 정의 변수
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/helm/template/MyChartStep1.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<br/><br/>

## 1. Helm 명령어

---


<figure>
  <img src="/img/helm/template/Command with Show, Template for Helm.jpg"
       alt="Command with Show, Template for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



<br/>

### 1-1) Chart Template 생성 


```sh
helm create mychart
```


### 1-2) Show 명령 


```sh
helm show values .
helm show chart .
helm show readme .
helm show all .
```


vi README.md

```sh
# Title
## Introduction
This is README.md
```

### 1-3) Template 명령


```sh
helm template mychart .
```

<br/>
<br/>

<figure>
  <img src="/img/helm/template/Command with Get, Install for Helm.jpg"
       alt="Command with Get, Install for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

<br/>

### 1-4) Chart 배포


```sh
helm install mychart .
```

### 1-5) Get 명령


```sh
helm get manifest mychart
helm get notes mychart
helm get values mychart
helm get all mychart
```

helm get values 비교

```sh
helm uninstall mychart
helm install mychart . -f values.yaml
helm install mychart . --set replicaCount=3
```

<br/>
<br/>




## 2. 내장 객체

---



<figure>
  <img src="/img/helm/template/Build in Objects for Helm.jpg"
       alt="Build in Objects for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>


### 2-1) configmap 추가 

vi cm-object.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: built-in-object
data:
  .Release: ______________________________________
  .Release.Name: {{ .Release.Name }}
  .Release.Namespace: {{ .Release.Namespace }}
  .Release.IsUpgrade: "{{ .Release.IsUpgrade }}"
  .Release.IsInstall: "{{ .Release.IsInstall }}"
  .Release.Revision: "{{ .Release.Revision }}"
  .Release.Service: {{ .Release.Service }}
  .Values: ______________________________________
  .Values.replicaCount: "{{ .Values.replicaCount }}"
  .Values.image.repository: {{ .Values.image.repository }}
  .Values.image.pullPolicy: {{ .Values.image.pullPolicy }}
  .Values.service.type: {{ .Values.service.type }}
  .Values.service.port: "{{ .Values.service.port }}"
  .Chart: ______________________________________
  .Chart.Name: {{ .Chart.Name }}
  .Chart.Description: {{ .Chart.Description }}
  .Chart.Type: {{ .Chart.Type }}
  .Chart.Version: {{ .Chart.Version }}
  .Chart.AppVersion: {{ .Chart.AppVersion }}
  .Template: ______________________________________
  .Template.BasePath: {{ .Template.BasePath }}
  .Template.Name: {{ .Template.Name }}
```



templates 폴더 안에 불필요한 파일 삭제 

```sh
rm -rf deployment.yaml hpa.yaml ingress.yaml serviceaccount.yaml tests
```


install 배포

```sh
helm install mychart . -n default
helm get manifest mychart 
helm upgrade mychart . -n default
```



<br/>
<br/>



## 3. 변수 주입 우선순위

---


<figure>
  <img src="/img/helm/template/values yaml for Helm.jpg"
       alt="values yaml for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>


### 3-1) configmap 추가

vi cm-values.yaml


```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: values-yaml
data:
  env: {{ .Values.configMapData.env }}
  log: {{ .Values.configMapData.log }}
  path: "{{ .Values.configMapData.path }}"
```

### 3-2) values.yaml에 해당 속성 추가
```yaml
configMapData:
  env: dev
  log: debug
  path: "/data"
```

### 3-3) 확인

```sh
helm template mychart .
```


### 3-4) Prod용 values_prod.yaml 파일 추가

vi values_prod.yaml

```yaml
configMapData:
  env: prod
  log: info
```


### 3-5) -f 옵션 적용

```sh
helm template mychart . -f ./values_prod.yaml
```

### 3-6) -f 옵션 적용, --set 옵션 적용

```sh
helm template mychart . -f ./values_prod.yaml --set configMapData.log=debug
```

<br/>
<br/>



## 4. 사용자 정의 변수

---

<figure>
  <img src="/img/helm/template/helpers tpl for Helm.jpg"
       alt="helpers tpl for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



_helpers.tpl 파일

```sh
{{/*
Expand the name of the chart.
*/}}
{{- define "mychart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "mychart.labels" -}}
helm.sh/chart: {{ include "mychart.chart" . }}
{{ include "mychart.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}
```

<br/>

service.yaml 파일

```sh
apiVersion: v1
kind: Service
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
```

```sh
helm template mychart .
```
<br/>
<br/>




## Referenece
---

### __Helm__
  - __차트 템플릿에 관한 빠른 가이드__ : <https://helm.sh/docs/chart_template_guide/getting_started/>
  - __내장 객체__ : <https://helm.sh/docs/chart_template_guide/builtin_objects/>
  - __변수주입과 우선순위__ : <https://helm.sh/docs/chart_template_guide/values_files/>
  - __사용자 정의 변수__ : <https://helm.sh/docs/chart_template_guide/named_templates/>
  - __Helm 생성시 명명규칙__ : <https://helm.sh/docs/chart_best_practices/conventions/>
  
### __Others__
  - __Markdown 가이드__ : <https://www.markdownguide.org/basic-syntax/>
  - __GoDoc 패키지 템플릿__ : <https://godoc.org/text/template>  
  - __Sprig 함수__ : <https://masterminds.github.io/sprig/>
  
  