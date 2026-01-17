# slurm-on-eks


본 워크숍은 Amazon EKS 환경에서 Slurm(Simple Linux Utility for Resource Management)을 활용하여, Llama 3와 같은 대규모 모델을 가장 비용 효율적이고 안정적으로 학습시키는 분산 트레이닝 기법에 대해 설명합니다. 전통적인 HPC(고성능 컴퓨팅) 스케줄러인 Slurm은 자원이 100% 확보되어야 작업을 시작하는 '갱 스케줄링(Gang Scheduling)'을 기본적으로 제공하고 있어 대규모 GPU 자원을 관리할 때 쿠버네티스의 유연함과 Slurm의 안정성을 결합하는 것이 매우 중요합니다. Slinky는 쿠버네티스의 작업 요청을 Slurm에 전달하고, 할당된 자원을 포드(Pod)에 정밀하게 매핑하는 가교 역할을 수행합니다. 이 워크샵은 Slinky 를 이용하여 쿠버테티스 환경에서 GPU 자원을 투명하게 사용하는 방식을 다룹니다. 이를 통해 참가자들은 별도의 복잡한 설정 없이도 익숙한 쿠버네티스 환경 내에서 Slurm의 강력한 스케줄링 성능을 마스터하고, 최적의 인프라 효율을 구현하는 실전 노하우를 습득할 수 있습니다.


* http://ec2-43-203-227-222.ap-northeast-2.compute.amazonaws.com:9090


### _Topics_ ###
  
* [C1. VPC 생성하기](https://github.com/gnosia93/slurm-on-k8s/tree/main/tf)
  
* [C2. EKS 클러스터 생성하기](https://github.com/gnosia93/slurm-on-k8s/blob/main/lession/2-building-eks-cluster.md)

* [C3. slinky 설치하기](https://github.com/gnosia93/slurm-on-k8s/blob/main/lession/3-install-slinky.md)

* [C4. slurm 시작하기](https://github.com/gnosia93/slinky-on-k8s/blob/main/lession/4-get-started-slurm.md)

* [C5. 파티션 설정하기](https://github.com/gnosia93/slurm-on-eks/blob/main/lession/5-setup-partition.md)

* [C6. 복원력 및 작업 재시작](https://github.com/gnosia93/slurm-on-eks/blob/main/lession/6-resilliency.md)

* [C7. 작업 모니터링 하기]

* [C8. 체크 포인트 저장하기]
    
* 노드 간 통신 병목을 해결하여 GPU 사용률(Utilization)을 90% 이상으로 끌어올리는 방법.
```
성능 확인 방법
학습 중 노드에 접속하여 아래 명령어로 확인해 보세요.
bash
# GPU 사용률 및 전력 소모 확인 (전력 소모가 높을수록 일을 많이 하는 중)
nvidia-smi dmon -s uc

# EFA 통신량 확인
watch -n 1 "fi_info -p efa"
```

* 내용: Slurm의 Job ID 기반으로 어떤 팀(또는 사용자)이 자원을 얼마나 썼는지 계산하고, 특정 예산을 초과하면 작업을 강제 종료하거나 알림을 보내는 정책 설정.
* 핵심: Karpenter의 Termination 정책과 연동하여 '좀비 노드' 방지하기.
* 내용: AWS FSx for Lustre를 Slurm 노드에 마운트하여 데이터 로딩 속도와 체크포인트 저장 속도를 극대화하는 법.
---

1. Pod 기반 로그인 노드 (가장 간편한 방식)
EKS 내부에 slurm-client용 Pod을 하나 띄워두고 kubectl exec로 접속해 사용하는 방식입니다. Slinky Helm 차트에 기본 포함되어 있는 경우가 많습니다.

```
loginNode:
  enabled: true
  image: "slinky-slurm-client:latest"
  # FSx 마운트 설정을 공유하여 로그인 노드에서도 데이터를 볼 수 있게 함
  extraVolumeMounts:
    - name: fsx-storage
      mountPath: /shared
```

```
kubectl exec -it deployment/slurm-login-node -n slurm -- /bin/bash

```
### __ref__ ##
* https://www.youtube.com/watch?v=Vi8chqAFuN0
