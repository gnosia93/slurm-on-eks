* Raw 데이터를 FSx 위에서 병렬로 읽어 토크나이징(Tokenizing)하거나 TFRecord/WebDataset 형태로 변환한 뒤, 동일한 FSx 경로에 저장.
* 성능 팁: Lustre 스트라이핑(Striping) 설정을 통해 대용량 파일을 여러 객체 저장소에 분산 저장하면 대규모 GPU 학습 시 읽기 성능이 극대화.
