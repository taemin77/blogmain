
---
title: "Authorization"
linkTitle: "Authorization"
weight: 4
date: 2021-08-02
description: > 
  RBAC, Role, RoleBinding
---

<div class="mx-auto">
	<a class="btn btn-lg btn-secondary mr-3 mb-4" href="/documents/intermediate/Authorization.pdf" download>
		강의자료 PDF <i class="fas fa-download ml-2"></i>
	</a>
</div>


<figure>
  <img src="/img/practice/intermediate/Access API with Authenticaiton for Kubernetes.jpg"
       alt="Access API with Authenticaiton for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


<br/>
<br/>

<figure>
  <img src="/img/practice/intermediate/Access API with Authorization RBAC for Kubernetes.jpg"
       alt="Access API with Authorization RBAC for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>


## 1. 자신의 Namespace 내에 Pod들만 조회할 수 있는 권한
---

<figure>
  <img src="/img/practice/intermediate/Access API with Authorization Role RoleBinding1 for Kubernetes.jpg"
       alt="Access API with Authorization Role RoleBinding1 for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>

### 1-1) Role 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: r-01
  namespace: nm-01
rules:
- apiGroups: [""]
  verbs: ["get", "list"]
  resources: ["pods"]
```


### 1-2) RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: rb-01
  namespace: nm-01
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: r-01
subjects:
- kind: ServiceAccount
  name: default
  namespace: nm-01
```

### 1-3) Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-1
spec:
  selector:
    app: pod
  ports:
  - port: 8080
    targetPort: 8080
```


### 1-4) Https API 호출 (Token)

case1) postman

`header` Authorization : Bearer TOKEN

```sh
https://192.168.0.30:6443/api/v1/namespaces/nm-01/pods/
```



case2) curl

```sh
curl -k -H "Authorization: Bearer TOKEN" https://192.168.0.30:6443/api/v1/namespaces/nm-01/pods/
```


<br/>
<br/>



## 2. 모든 Namespace 내에 Object들에 대해 모든 권한을 부여

---

<figure>
  <img src="/img/practice/intermediate/Access API with Authorization Role RoleBinding2 for Kubernetes.jpg"
       alt="Access API with Authorization Role RoleBinding2 for Kubernetes."
       class="mt-3 mb-3 border border-info rounded" />
</figure>



### 2-1) Namespaces 

```sh
apiVersion: v1
kind: Namespace
metadata:
  name: nm-02
```



### 2-2) ServiceAccount 

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: sa-02
  namespace: nm-02
```


### 2-3) ClusterRole 

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cr-02
rules:
- apiGroups: ["*"]
  verbs: ["*"]
  resources: ["*"]
```


### 2-4) ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: rb-02
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cr-02
subjects:
- kind: ServiceAccount
  name: sa-02
  namespace: nm-02
```



### 2-5) Https API 호출 (Token)


case1) postman

`header` Authorization : Bearer TOKEN

```sh
https://192.168.0.30:6443/api/v1/namespaces/nm-01/service
```


case2) curl


```sh
curl -k -H "Authorization: Bearer TOKEN" https://192.168.0.30:6443/api/v1/namespaces/nm-01/service
```

<br/>
<br/>





## Referenece
---
### __Kubernetes__
  - __Using RBAC Authorization__ : <https://kubernetes.io/docs/reference/access-authn-authz/rbac/>
  - __Review Your Request Attributes__ : <https://kubernetes.io/docs/reference/access-authn-authz/authorization/#review-your-request-attributes>
  
