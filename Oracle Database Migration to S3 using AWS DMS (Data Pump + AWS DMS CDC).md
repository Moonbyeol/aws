
# 개요

Oracle Database의 데이터를 하이브리드 방식으로 진행

1. 초기 데이터 적재는 `Oracle Data Pump`를 통해 `.dmp` 파일을 생성하고 S3로 업로드하여 대용량 데이터를 빠르게 이전
    
2. 그 후 변경 데이터(CDC)는 `AWS Database Migration Service (DMS)`를 활용하여 Oracle과 S3 간의 지속적인 데이터 동기화

이를 통해 안정적이고 효율적인 데이터 레이크 환경을 구축


# 1. Oracle Data Pump

- Oracle 버전: 10g 이상 (`expdp` 지원)
- Oracle 계정에 `DBA` 권한 또는 `EXP_FULL_DATABASE` 롤 필요
- `DIRECTORY` 객체 생성 및 접근 권한 필요
- S3 업로드를 위한 AWS CLI 설정 (Oracle 서버 또는 별도 머신에서)


## 1) directory 객체 생성


```sql
mkdir -p /home/oracle/dpump
```

```sql
CREATE OR REPLACE DIRECTORY DATA_PUMP_DIR AS '/home/oracle/dpump';
GRANT READ, WRITE ON DIRECTORY DATA_PUMP_DIR TO myuser;
```


## 2) 덤프 파일 생성

```bash
expdp system/oracle \
directory=DATA_PUMP_DIR \ 
dumpfile=oracle_full.dmp \
logfile=oracle_full.log \
full=y 
```
- expdp : DBA 권한을 가진 계정으로 실행(이이디/비번)
- full=y : 전체 DB를 대상으로 export

## 3) 파일 압축 (선택)

```bash
gzip export_data.dmp
```

## 4) 파일 업로드
```bash
aws s3 cp export_data.dmp.gz s3://my-bucket/oracle-dumps/
```
- s3://my-bucket/oracle-dumps/ 경로 확인

