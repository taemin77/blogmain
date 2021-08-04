
---
title: "Prometheus 차트 분석"
linkTitle: "Prometheus"
weight: 2
date: 2021-08-02
description: >
  
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/helm/sample/Prometheus.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<br/><br/>



## 1. Prometheus 차트 설치 및 배포

---

### 1-1) 레포지토리 등록

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

### 1-2) Tomcat 7.1.2 다운로드

```sh
helm pull prometheus-community/prometheus --version 13.8.0
tar -xf prometheus-13.8.0.tgz
```

### 1-3) Tomcat Template 보기

```sh
helm template mytomcat .
```


<br/>
<br/>



## 2. Prometheus에서 나오는 함수 따라해보기

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
colors: 
  - "blue"
  - "red"
  - "green" 

server: 
  path:
    prefix: "prom"
```


### 2-3) configmap 추가 

vi cm.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: tomcat
data:
#<splitList>
 {{- $url := splitList "/" "helm.sh/docs/helm/" }}
  splitList:
    host: {{ first $url }}
    path: {{ rest $url | join "/" }}

#<regexMatch>
  regexmatch: {{ regexMatch ".*\\.ya?ml$" "config.yaml" }}

#<index>
  index:
    colors: "{{ index .Values.colors 0 }}"
 {{- if index .Values "server" "path" "prefix" }}
    path: /{{ index .Values "server" "path" "prefix" }}/ready
 {{- end }}

#<semver>
{{- $version := semver "1.2.3-alpha.1+123" }} 
  semver:
    major: {{ $version.Major }}
    minor: {{ $version.Minor }}
    patch: {{ $version.Patch }}
    prerelease: {{ $version.Prerelease }}
    metadata: {{ $version.Metadata }}
    original: {{ $version.Original }}

#<semverCompare>
  semverCompare:
    greaterthan: {{ semverCompare ">1.2.1" "1.2.3" }}
    lessthan: {{ semverCompare "<1.2.1" "2.2.3" }}
    equal: {{ semverCompare "=1.2.*" "1.2.3" }}   #1.2.0 <= [1.2.*] < 1.3.0
    tilde: {{ semverCompare "~1.2.3" "1.2.0" }}   #1.2.3 <= [~1.2.3] < 1.3.0
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
  - __차트 종속성__ : <https://helm.sh/docs/chart_best_practices/dependencies/>
  - __Semantic Versions__ : <https://github.com/Masterminds/semver>
  - __함수__ : <https://helm.sh/docs/chart_template_guide/function_list/>
  