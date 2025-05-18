
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
- s3://my-bucket/oracle-dumps/ 경로 확인



# DMS CDC
- 

## 1) IAM 역할 만들기
- IAM 콘솔
- 역할 생성
	- aws 서비스
	- 사용 사례 : DMS

	- 권한
		- AmazonS3FullAccess

	- 역할 이름 : dms-s3-role
	- 생성


## 2) Oracle DB log 준비

```sql
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
```

## 3) DMS replication instance 만들기
- DMS 콘솔
- replication instance 생성
	- 이름 : oracle-to-s3-replication
	- 인스턴스 클래스 : 선택
	- 스토리지 : 선택
	- VPC : VPN이 연결된 VPC 선택
	- 퍼블릭 엑세스 : 끄기 (사설망만 사용)
	- 나머지 기본값
	- 생성


## 4) Oracle Endpoint 생성
- DMS 콘솔
- Endpoint 생성
	- endpoint type : Source
	- 식별자 : oracle-source-endpoint
	- 소스 엔진 : oracle
	- 엔드포인트 데이터베이스에 엑세스 : 수동으로 엑세스 정보 제공
		- 서버 이름 : oracle 서버 내부망 ip
		- 포트 : 1521
		- 사용자 이름/비밀번호 : oracle 계정
		- SID : `ORCL` 또는 `XE` (둘 중 실제 사용 중인 것)
	- 연결 테스트
		- 위에서 생성한 dms instance 선택
	- 성공 해야 함. 하면 다음으로


## 5) S3 Endpoint 생성
- DMS 콘솔
- Endpoint 생성
	- endpoint type : target
	- 식별자 : st-target-endpoint
	- 소스 엔진 : Amazon S3
	- Amazon S3 bucket : 버킷 선택
	- IAM role for S3 bucket : Use ana existing IAM role 선택
	- 위에서 만들었던 IAM role 선택

## 6) Migration Task 생성
- DMS 콘솔
- Database migration task 생성
	- 식별자 : oracle-cdc-to-s3-task
	- replication instance : 위에서 생성한 인스턴스
	- source database endpoint : oracle-source-endpoint
	- target database endpoint : s3-target-endpoint
	- mygration type : CDC
	- task 설정
		- 대부분 기본값
		- 데이터 검증 - 필요시
		- 테스크 로그 - 필요시
	- table mappings
		- 새 선택 규칙 추가
		- 
