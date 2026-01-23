## Slurm 이란 ##

Slurm은 Simple Linux Utility for Resource Management의 약자이다. 
이름은 'Simple'이라고 시작하지만, 실제로는 전 세계 슈퍼컴퓨터와 AI 클러스터에서 가장 많이 사용되는 오픈 소스 워크로드 관리자(Workload Manager)이자 작업 스케줄러이다.

### Slurm의 3가지 핵심 역할 ##
* 리소스 할당 (Allocate): 사용자가 요청한 시간 동안 특정 개수의 CPU, GPU, 메모리 등의 자원을 독점적으로 할당해 줍니다.
* 작업 실행 (Run): 할당된 노드에서 실제 계산 작업(예: AI 학습 모델)을 시작하고 실행 상태를 감시합니다.
* 대기열 관리 (Queue): 수많은 사용자가 동시에 작업을 던질 때, 우선순위(Priority)에 따라 줄을 세우고 자원이 나면 순서대로 투입합니다


## Accounting ##
Slurm에서 어카운팅(Accounting)은 시스템 자원을 "누가, 언제, 얼마나" 사용했는지 기록하고 관리하는 핵심 감사 시스템으로
단순한 기록을 넘어, 우선순위(Fairshare)를 결정하는 근거 데이터가 되기 때문에 매우 중요하다.

### 1. 어카운팅의 핵심 구조: SlurmDBD ###
* 어카운팅은 SlurmDBD(Slurm Database Daemon)라는 별도의 데몬과 MySQL/MariaDB를 통해 작동합니다.
* 사용자(User): 작업을 던지는 개별 계정.
* 어카운트(Account): 팀이나 프로젝트 단위 (예: ai_team, hpc_project). 사용자는 여러 어카운트에 속할 수 있습니다.
* 연합(Association): [사용자 + 어카운트 + 파티션 + 클러스터]를 하나로 묶는 논리적 단위입니다.

### 2. 어카운팅으로 할 수 있는 것 (Consultative Points) ###
* 자원 사용량 추적: 특정 프로젝트가 이번 달에 GPU를 몇 시간(GPU-hours) 사용했는지 정확히 계산하여 과금(Billing)하거나 보고서를 뽑을 수 있다.
* 우선순위(Fairshare) 계산: 어카운팅 데이터에 기록된 과거 사용량(Historical Usage)을 바탕으로, 최근에 자원을 많이 쓴 사용자의 우선순위는 낮추고, 덜 쓴 사용자는 높여주는 공평한 스케줄링을 구현한다.
* 할당량 제한(Resource Limits): 특정 어카운트가 동시에 사용할 수 있는 GPU 개수나 최대 CPU 시간을 설정하여 자원 독점을 방지한다.

### 3. 주요 명령어 ###
* sacct: 종료된 작업의 상세 기록(사용 시간, 메모리 피크 등)을 조회
* sacctmgr: 사용자, 어카운트, 쿼터(Quota) 등을 관리하는 관리자용 도구
* sreport: 주간/월간 자원 이용률 보고서를 생성.

### 4. 거버넌스 ###
#### 1. 어카운트(그룹) 생성 ####
먼저 사용자가 소속될 프로젝트나 팀 단위의 어카운트를 만듭니다. 이때 어카운트 레벨에서 전체 쿼터를 미리 걸 수도 있습니다.
* 명령어: sacctmgr add account <어카운트명> Description="<설명>" Organization="<조직명>"
* 예시: sacctmgr add account ai_project Description="AI_Team_Project" Organization="Research_Dept"

#### 2. 사용자 생성 및 어카운트 연결 ####
사용자를 특정 어카운트에 귀속시키며 생성합니다.
* 명령어: sacctmgr add user <사용자명> Account=<어카운트명>
* 예시: sacctmgr add user gildong Account=ai_project

#### 3. 쿼터(Resource Limits) 설정 ####
이제 핵심인 쿼터를 부여합니다. 특정 사용자 혹은 어카운트 전체에 대해 CPU, GPU, 실행 가능 작업 수 등을 제한할 수 있습니다. 
* GPU 개수 제한 (사용자 단위):
```
sacctmgr modify user gildong set GrpTRES=gres/gpu=2
```
(설명: gildong 사용자가 동시에 사용할 수 있는 GPU 총합을 2개로 제한)

* 최대 실행 작업 수 제한 (어카운트 단위):
```
sacctmgr modify account ai_project set MaxJobs=10
```
(설명: ai_project 팀 전체가 동시에 돌릴 수 있는 작업 수를 10개로 제한)

* CPU 시간 제한 (어카운트 단위):
```
sacctmgr modify account ai_project set GrpCPURunMins=10000
```
(설명: 해당 팀이 사용 중인 총 CPU 시간 합계를 제한)

* 설정값 확인
```
sacctmgr show association user=<사용자명> format=Account,User,Partition,GrpTRES,MaxJobs
```

---

* Slurm을 이용한 GPU 전용 파티션 설정법
* 작업 간의 우선순위(Fairshare) 관리 전략
* Slurm과 Docker/Singularity 컨테이너 연동 방식
* 직접적인 명령어(sbatch, squeue 등) 활용법
