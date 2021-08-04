
---
title: "Grafana 차트 분석"
linkTitle: "Grafana"
weight: 3
date: 2021-08-02
description: >
  Helm Test
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/helm/sample/Grafana.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>

<br/><br/>



## 1. Grafana 차트 설치 및 배포

---

### 1-1) 레포지토리 등록

```sh
helm repo add grafana https://grafana.github.io/helm-charts
```

### 1-2) Grafana 6.9.1 다운로드

```sh
helm pull grafana/grafana --version 6.9.1
tar -xf grafana-6.9.1.tgz
```

### 1-3) Grafana Template 보기

```sh
helm template my-grafana .
```

### 1-4) Grafana Install 배포

```sh
helm install my-grafana .
```

### 1-5) Grafana Test 하기

```sh
helm test my-grafana
```


<br/>
<br/>


## Referenece
---

### __Helm__
  - __차트 테스트__ : <https://helm.sh/docs/topics/chart_tests/>
  - __Helm Test Command__ : <https://helm.sh/docs/helm/helm_test/>
  - __내장객체(Files)__ : <https://helm.sh/docs/chart_template_guide/builtin_objects/>
  