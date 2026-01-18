* Raw 데이터를 FSx 위에서 병렬로 읽어 토크나이징(Tokenizing)하거나 TFRecord/WebDataset 형태로 변환한 뒤, 동일한 FSx 경로에 저장.
* 성능 팁: Lustre 스트라이핑(Striping) 설정을 통해 대용량 파일을 여러 객체 저장소에 분산 저장하면 대규모 GPU 학습 시 읽기 성능이 극대화.


```
#!/bin/bash
#SBATCH --job-name=tokenize_data
#SBATCH --array=0-99             # 100개의 병렬 태스크 생성
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=16       # 각 태스크당 16개 코어 사용

# Task ID에 따라 데이터 샤드 지정
INPUT_FILE="/data/raw/shard_${SLURM_ARRAY_TASK_ID}.txt"
OUTPUT_DIR="/data/processed/shard_${SLURM_ARRAY_TASK_ID}"

# 토크나이징 스크립트 실행 (예: HuggingFace Tokenizers)
python tokenize_script.py --input $INPUT_FILE --output $OUTPUT_DIR --format webdataset
```



### FSx for Lustre 최적화 팁 (Striping) ###
대용량 WebDataset 파일을 생성할 때, Lustre의 Striping 설정을 통해 파일 하나를 여러 서버에 분산 저장하면 학습 시 읽기 속도가 수 배 빨라진다. 
```
# 결과 폴더에 스트라이핑 설정 (파일을 8개의 저장소에 분산)
lfs setstripe -c 8 /data/processed
```

### S3로 최종 결과물 백업 (Export) ###
전처리가 완료된 WebDataset은 다시 S3로 영구 저장해야 합니다. AWS 데이터 리포지토리 연동(DRA) 기능을 사용하면 FSx에서 생성된 파일을 즉시 S3 버킷으로 밀어 넣을 수 있다.
