
---
title: "Tomcat 차트 분석"
linkTitle: "Tomcat"
weight: 1
date: 2021-08-02
description: >
  
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/helm/sample/Tomcat.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<br/><br/>



## 1. tomcat 차트 설치 및 배포

---

### 1-1) 레포지토리 등록

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
```

### 1-2) Tomcat 7.1.2 다운로드

```sh
helm pull bitnami/tomcat --version 7.1.2
tar -xf tomcat-7.1.2.tgz
```

### 1-3) Tomcat Template 보기

```sh
helm template mytomcat .
```


<br/>
<br/>



## 2. tomcat에서 나오는 함수 따라해보기

---

### 2-1) Chart Template 생성 


```sh
helm create mychart
```
templates 폴더 안에 불필요한 파일 삭제 

```sh
rm -rf deployment.yaml hpa.yaml ingress.yaml serviceaccount.yaml service.yaml tests
```

### 2-2) test-values.yaml에 해당 속성 추가

vi test-values.yaml

```yaml
data:

include : "value2"

typeIs1 : "text"
typeIs2 : true

defaultLevel : info
dev:
  env: dev
  log: "{{ .Values.defaultLevel }}"
qa:
  env: qa
  log: "{{ .Values.defaultLevel }}"

data1:
data2:
data3: "text3"
```


### 2-3) configmap 추가 

vi cm.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tomcat
data:
#<dict>
 {{- $myDict := dict "key1" "value1" }}
  dict: {{ get $myDict "key1" }}

#<include>
  include1: {{- include "mychart.include" (dict "key1" "value1") | nindent 4 }}
  include2: {{- include "mychart.include" (dict "key1" .Values.include) | nindent 4 }}

#<typels>
  {{- if typeIs "string" .Values.typeIs1 }}
  typels1: {{ .Values.typeIs1 }}
  {{- end }}
  {{- if typeIs "bool" .Values.typeIs2 }}
  typels2: {{ .Values.typeIs2 }}
  {{- end }}

#<tpl>
  normal: "{{ .Values.dev.log }}"
  tpl: "{{ tpl .Values.dev.log . }}"

#<coalesce>
  coalesce1: {{ coalesce .Values.data1 .Values.data2 "text1" }}
  coalesce2: {{ coalesce .Values.data1 "text2" .Values.data3 }}
```


### 2-4) Template 명령

```sh
helm template mychart ./../ -f ./../test-values.yaml
```


<br/>
<br/>


## Referenece
---

### __Helm__
  - __함수__ : <https://helm.sh/docs/chart_template_guide/function_list/>
  - __tpl 함수__ : <https://helm.sh/docs/howto/charts_tips_and_tricks/#using-the-tpl-function>
  - __지역변수__ : <https://helm.sh/docs/chart_template_guide/variables/>
  