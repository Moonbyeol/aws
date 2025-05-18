
대용량 데이터 s3에 업로드
glue나 athena 에서 바로 사용 불가....

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
- expdp : DBA 계정 or `EXP_FULL_DATABASE` 롤을 가진 계정으로 실행(이이디/비번)
- full=y : 전체 DB를 대상으로 export

## 3) 파일 압축 (선택)

```bash
gzip export_data.dmp
```

## 4) 파일 업로드
```bash
aws s3 cp export_data.dmp.gz s3://my-bucket/oracle-dumps/
```
- 버킷 경로 확인


