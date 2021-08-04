
---
title: "내 차트 만들기2"
linkTitle: "Step2"
weight: 2
date: 2021-08-02
description: >
  함수와 파이프라인, 제어 흐름, 여러가지 함수들, 지역변수
---


<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/helm/template/MyChartStep2.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<br/><br/>



## 1. 함수와 파이프라인

---

<figure>
  <img src="/img/helm/template/Function and Pipeline for Helm.jpg"
       alt="Function and Pipeline for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



<br/>

### 1-1) Chart Template 생성 


```sh
helm create mychart
```
templates 폴더 안에 불필요한 파일 삭제 

```sh
rm -rf deployment.yaml hpa.yaml ingress.yaml serviceaccount.yaml service.yaml tests
```

### 1-2) test-values.yaml에 해당 속성 추가
```yaml
func:
  enabled: true
  
pipe:
  log: info
```


### 1-3) configmap 추가 

vi cm1.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: function-and-pipeline
data:
  Function_Argument:
    quote: {{ quote .Values.func.enabled }}     # quote(arg1)
    include: {{ include "mychart.name" . }}  # include(arg1, arg2)

  Function_Quote:
    function_case1: {{ .Values.func.enabled }}
    function_case2: {{ quote .Values.func.enabled }}
    function_case3: "{{ .Values.func.enabled }}"

  Pipeline:
    upper: {{ .Values.pipe.log | upper }}
    upper.repeat: {{ .Values.pipe.log | upper | repeat 2 }}
    upper.repeat.quote: {{ .Values.pipe.log | upper | repeat 2 | quote }}
```

Template 명령

```sh
helm template mychart ./../ -f ./../test-values.yaml
```


<br/>
<br/>



## 2. 흐름 제어

---

<figure>
  <img src="/img/helm/template/Controller Flow with if, with, range for Helm.jpg"
       alt="Controller Flow with if, with, range for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>

### 2-1) test-values.yaml에 해당 속성 추가
```yaml
dev:
  env: dev
  log: info

qa:
  env: qa
  log: info
  
prod:
  env: prod
  log: 
  
data:
  - a
  - b
  - c
```


If 문에서 False 인 조건들
```sh
  Number:0, String: "", List: [], Object: {}, Boolean: false, Null
```
If 문에 쓰이는 함수들
```sh
  and(&&), or(||), ne(!=), not(!), eq(=), ge(>=), gt(>), le(<=), lt(<), default, empty 
```
         
<br/>
                                                                                                                                                   
### 2-2) IF 사용
                               
vi cm2-2.yaml            

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: flow-if
data:
  dev:
    env: {{ .Values.dev.env }}
    {{- if eq .Values.dev.env "dev" }}
    log: debug
    {{- else if .Values.dev.log }}
    log: {{ .Values.dev.log }}
    {{- else }}
    log: error
    {{ end }}

  qa:
    env: {{ .Values.qa.env }}
    {{- if eq .Values.qa.env "dev" }}
    log: debug
    {{- else if .Values.qa.log }}
    log: {{ .Values.qa.log }}
    {{- else }}
    log: error
    {{- end }}

  prod:
    env: {{ .Values.prod.env }}
    {{ if eq .Values.prod.env "dev" }}
    log: debug
    {{ else if .Values.prod.log }}
    log: {{ .Values.prod.log }}
    {{ else }}
    log: error
    {{ end }}
```


### 2-3) WITH 사용

vi cm2-3.yaml     

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: flow-with
data:
  dev:
  {{- with .Values.dev }}
    env: {{ .env }}
    log: {{ .log }}
  {{- end }}

  qa:
  {{- with .Values.qa }}
    env: {{ .env }}
    log: {{ .log }}
  {{- end }}
  
  prod:
  {{- with .Values.prod }}
    env: {{ .env }}
    log: {{ .log }}
  {{- end }}
```


### 2-4) RANGE 사용 

vi cm2-4.yaml     


```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: flow-range
data:
  yaml:
  {{- .Values.data | toYaml | nindent 2 }}

  range:
  {{- range .Values.data }}
  - {{ . }}
  {{- end }}

  range-quote:
  {{- range .Values.data }}
  - {{ . | quote }}
  {{- end }}

  range-upper-quote:
  {{- range .Values.data }}
  - {{ . | upper | quote }}
  {{- end }}
```

<br/>
<br/>





## 3. 여러가지 함수들 

---

<figure>
  <img src="/img/helm/template/Functions1 for Helm.jpg"
       alt="Functions1 for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

<figure>
  <img src="/img/helm/template/Functions2 for Helm.jpg"
       alt="Functions2 for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



<br/>

### 3-1) test-values.yaml에 해당 속성 추가

```yaml
print:
  a: "1"
  b: "2"

ternary: 
  case1: true
  case2: false
  
default:
  nil: 
  list: []
  object: {}
  number: 0
  string: ""
  boolean: false
```

### 3-2) Functions 사용 configmap 추가 

vi cm3.yaml          

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: string-func
data:
  print:
    print:  {{ print "Hard Cording" }}
    printf: {{ printf "%s-%s" .Values.print.a .Values.print.b }}

  ternary:
    case1: {{ .Values.ternary.case1 | ternary "1" "2" }}
    case2: {{ .Values.ternary.case2 | ternary "1" "2" }}

  indent:
    indent: 
{{ .Values.data | toYaml | indent 4 }}
    nindent1: {{ .Values.data | toYaml | nindent 4 }}
    nindent2:
    {{- .Values.data | toYaml | nindent 4 }}
          
  default: # Number:0, String: "", List: [], Object: {}, Boolean: false, Null
    nil:     {{ .Values.default.nil     | default "default" }}
    list:    {{ .Values.default.list    | default (list "default1" "default2") | toYaml | nindent 6}}
    object:  {{ .Values.default.object  | default "default:1" | toYaml | nindent 6 }}
    number:  {{ .Values.default.number  | default 1 }}
    string:  {{ .Values.default.string  | default "default" }}
    boolean: {{ .Values.default.boolean | default true }}

  trim:
    trim:       {{ trim "  hello " }}
    trimPrefix: {{ trimPrefix "-" "-hello" }}
    trimSuffix: {{ trimSuffix "-" "hello-" }}

  random:
    randAlphaNum: {{ randAlphaNum 5 }}   # 0-9a-zA-Z
    randAlpha:    {{ randAlpha 5 }}      # a-zA-Z
    randNumeric:  {{ randNumeric 5 }}    # 0-9
    randAscii:    {{ randAscii 5 }}      # ASCII characters

  trunc:    {{ trunc 5 "hello world" }}
  replace:  {{ "hello world" | replace " " "-" }}
  contains: {{ contains "cat" "catch" }}
  b64enc:   {{ b64enc "hello" }}
```




<br/>
<br/>




## 4. 지역변수

---

<figure>
  <img src="/img/helm/template/Local values with WITH for Helm.jpg"
       alt="Local values with WITH for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>


### 4-1) with 내부에서 사용할 경우

vi cm4-1.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: variables-with
data:
  dev:
  {{- $relname := .Release.Name -}}
  {{- with .Values.dev }}
    env: {{ .env }}
    release: {{ $relname }}
    log: {{ .log }}
  {{- end }}
```

<br/><br/>


<figure>
  <img src="/img/helm/template/Local values with RANGE for Helm.jpg"
       alt="Local values with RANGE for Helm."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>

### 4-2) range와 함께 사용

vi cm4-2.yaml


```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: variables-range
data:
  # for (int i ; i=list.length ; i++) { printf "i : list[i]" };
  index:
  {{- range $index, $value := .Values.data }}
    {{ $index }}: {{ $value }}
  {{- end }}

  # for (Map<key, value> map : list) { printf "map.key() : map.value()" };
  key-value:
  {{- range $key, $value := .Values.dev }}
    {{ $key }}: {{ $value | quote }}
  {{- end }}
```


<br/>
<br/>



## Referenece
---

### __Helm__
  - __함수와 파이프라인__ : <https://helm.sh/docs/chart_template_guide/functions_and_pipelines/>
  - __함수 리스트__ : <https://helm.sh/docs/chart_template_guide/function_list/>
  - __흐름제어__ : <https://helm.sh/docs/chart_template_guide/control_structures/>
  - __지역변수__ : <https://helm.sh/docs/chart_template_guide/variables/>
  