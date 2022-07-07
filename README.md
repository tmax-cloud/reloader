# reloader 가이드
### 현재 k8s는 configmap이나 secret이 업데이트 될 경우 해당 리소스를 사용하는 Deployment, Daemonsets, Statefulsets, Rollouts를 통해 생성된 pod 들의 재시작을 지원하지 않는다.

### 이에 대한 자동화 방안으로 3가지 방안이 있다.

1. configmap controller
2. stakater reloader
3. k8s-trigger-controller

configmap controller의 경우 configmap의 변화만 감지한다.

k8s-trigger-controller의 경우 configmap, secret의 변화 모두 감지하지만 deployment를 통해 생성된 pod만 재시작해주는 문제점이 있다.

stakater reloader는 configmap, secret의 변경사항을 모두 감지하며, 변경이 감지될 시 Deployment를 통해 생성된 pod뿐 아니라 Daemonsets, Statefulsets, Rollouts를 통해 생성된 pod 들의 재시작을 도와준다.

- k8s 1.19 이상의 버전에서만 지원한다.

## Install
```
helm install reloader -n reloader
```
## 사용법

### 1. auto

```yaml
kind: Deployment
metadata:
  annotations:
    reloader.stakater.com/auto: "true"
spec:
  template:
    metadata:
```

위와 같이 auto 설정을 줄 경우에는

현재 배포하려는 deployment에서 사용하고 있는 configmap 혹은 secret (환경 변수로 mount 되었는지, volume으로 mount 되었는지 여부와 상관없이)이 변경되었을 경우 자동으로 reload를 해주게 된다.

### 2. search, match

```yaml
kind: Deployment
metadata:
  annotations:
    reloader.stakater.com/search: "true"
spec:
  template:
```

다음과 같이 annotation을 설정할 경우 configmap 이나 secret에 아래와 같은 annotation이 설정된 리소스의 변경만 감지를 하고 reloading을 시켜준다.

```yaml
kind: ConfigMap
metadata:
  annotations:
    reloader.stakater.com/match: "true"
data:
  key: value
```

deployment annotation에 search와 auto는 같이 사용할 수 없다.

 

### 3. 특정 configmap의 변경사항만 탐지하여 reloading을 하고 싶을때

```yaml
kind: Deployment
metadata:
  annotations:
    **configmap**.reloader.stakater.com/reload: "컨피그맵 명"
spec:
  template:
    metadata:
```

다수의 configmap을  지정하여 변경하고 싶을시에는 , 로 구분 지어서 나열한다.

```yaml
kind: Deployment
metadata:
  annotations:
    **configmap**.reloader.stakater.com/reload: "foo-configmap,bar-configmap,baz-configmap"
spec:
  template: 
    metadata:
```

### 4. 특정 secret의 변경사항만 탐지하여 reloading을 하고 싶을때

```yaml
kind: Deployment
metadata:
  annotations:
    **secret**.reloader.stakater.com/reload: "시크릿 명"
spec:
  template: 
    metadata:
```

다수의 secret을  지정하여 변경하고 싶을시에는 , 로 구분 지어서 나열한다.

```yaml
kind: Deployment
metadata:
  annotations:
    **secret**.reloader.stakater.com/reload: "foo-secret,bar-secret,baz-secret"
spec:
  template: 
    metadata:
```
