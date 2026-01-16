1. slogin 포드 접속하기
먼저 명령어를 입력할 수 있는 관문인 slogin 포드의 이름을 확인하고 접속합니다.
```
kubectl get pods -n slurm
```
[결과]


```
kubectl exec -it <slogin-pod-name> -n slurm -- /bin/bash
```
