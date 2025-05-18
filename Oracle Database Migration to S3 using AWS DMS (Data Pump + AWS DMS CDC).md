
# 개요

Oracle Database의 데이터를 하이브리드 방식으로 진행

1. 초기 데이터 적재는 `Oracle Data Pump`를 통해 `.dmp` 파일을 생성하고 S3로 업로드하여 대용량 데이터를 빠르게 이전
    
2. 그 후 변경 데이터(CDC)는 `AWS Database Migration Service (DMS)`를 활용하여 Oracle과 S3 간의 지속적인 데이터 동기화

이를 통해 안정적이고 효율적인 데이터 레이크 환경을 구축


# 1. Oracle Data Pump

### 1) directory 객체 생성
```sql
CREATE OR REPLACE DIRECTORY DATA_PUMP_DIR AS '/home/oracle/dpump';
GRANT READ, WRITE ON DIRECTORY DATA_PUMP_DIR TO myuser;
```

