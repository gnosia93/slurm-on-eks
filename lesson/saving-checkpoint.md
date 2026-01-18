모델이 커질수록 쓰기 속도가 병목이 됩니다. 여기서 **PyTorch FSDP의 분산 체크포인트(Distributed Checkpoint)**와 **FSx 스트라이핑(lfs setstripe)**의 시너지를 확인하는 내용을 추천합니다.

### 1. FSx for Lustre 생성 및 S3 데이터 연결 ###
FSx for Lustre를 생성할 때 학습 데이터가 담긴 S3 버킷을 데이터 리포지토리(Data Repository)로 연결합니다. 
* Lazy Loading: 모든 데이터를 미리 다운로드할 필요 없이, 학습 스크립트가 파일에 접근하는 순간 S3에서 Lustre 캐시로 데이터를 즉시 가져옵니다.
* 성능 설정: p4dn의 성능을 다 쓰려면 SSD 타입과 최소 200 MB/s/TiB 이상의 처리량(Throughput) 설정을 추천합니다.

### 2. Slinky 환경에서 Slurm 노드에 마운트하기 ###
Slinky(EKS 기반)에서는 FSx for Lustre CSI Driver를 사용하여 PersistentVolume(PV)으로 마운트합니다.

* Helm values.yaml 설정 (Pod 레벨 마운트)
Slinky 파티션 정의에서 해당 스토리지를 볼륨으로 추가합니다.
```
clusters:
  - name: "slinky-cluster"
    partitions:
      - name: "gpu-partition"
        # ... 기존 설정 ...
        extraVolumes:
          - name: fsx-storage
            persistentVolumeClaim:
              claimName: fsx-pvc
        extraVolumeMounts:
          - name: fsx-storage
            mountPath: /shared  # Slurm 노드 내부에서 보일 경로

```

* 체크포인트 저장 속도 극대화 (Write-through)
Llama 3 70B 모델은 체크포인트 용량이 수백 GB에 달합니다.
* 병렬 쓰기: FSx for Lustre는 수백 개의 프로세스가 동시에 하나의 파일 시스템에 쓰는 병렬 I/O에 최적화되어 있습니다.
* S3 자동 업로드: 학습 중 /shared/checkpoints에 저장된 파일은 설정에 따라 S3로 비동기 자동 업로드되도록 구성할 수 있습니다. 이를 통해 노드 장애 시에도 S3에 안전하게 데이터가 남습니다. S3 내보내기 설정(AWS)을 참고하세요.

[슬럽 스크립트]
```
#SBATCH --job-name=llama3_fsx
# ... 중략 ...

# 데이터 읽기 속도를 위해 Lustre 경로 지정
DATA_DIR="/shared/dataset/llama3_raw"
CHECKPOINT_DIR="/shared/checkpoints/$SLURM_JOB_ID"

srun torchrun ... \
    --data_path $DATA_DIR \
    --save_dir $CHECKPOINT_DIR
```

실무 꿀팁: 스트라이핑(Striping) 설정
파일이 매우 클 경우(체크포인트 파일 등), Lustre의 Striping 기능을 쓰면 여러 스토리지 노드에 데이터를 분산 저장하여 성능을 더 끌어올릴 수 있습니다.
bash

#### /shared/checkpoints 디렉토리에 생성되는 파일들을 8개의 스토리지 노드에 분산 저장
```
lfs setstripe -c 8 /shared/checkpoints
```
