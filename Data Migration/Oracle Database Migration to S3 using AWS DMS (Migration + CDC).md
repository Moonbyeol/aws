
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
	- mygration type : Migration and CDC
	- task 설정
		- 시작 모드, 시작 시점 설정 필요
		- 데이터 검증 - 필요시
		- 테스크 로그 - 필요시
	- table mappings
		- 새 선택 규칙 추가
			- 스키마 입력 클릭
			- schema name : % (전체 선택 의미)
			- table name : % (전체 선택 의미, 다른 것들은 [참고](https://docs.aws.amazon.com/ko_kr/dms/latest/userguide/CHAP_Tasks.CustomizingTasks.TableMapping.SelectionTransformation.Wildcards.html))
			- include
	- 생성
## 7) 테스트



