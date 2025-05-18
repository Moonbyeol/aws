
## 1) Oracle DB log 활성화

```sql
ALTER DATABASE ADD SUPPLEMENTAL LOG DATA;
```
- DBA 권한이 부여된 사용자 계정으로 해야 함

## 2) S3 버킷 생성
- 버킷 생성

## 3) IAM 역할 생성 (최소 권한)

### 3-1. 사용자 정의 정책 생성
- IAM 콘솔
- 정책 생성
	- 정책 이름 : DMS-S3-MinimalPolicy
	- json
		```json
		 {
		"Version": "2012-10-17",
		"Statement": [
		 {
		   "Sid": "S3MinimalAccessForDMS",
		   "Effect": "Allow",
		   "Action": [
		     "s3:PutObject",
		     "s3:GetObject",
		     "s3:DeleteObject",
		     "s3:ListBucket"
		   ],
		   "Resource": [
		     "arn:aws:s3:::oracle-cdc-bucket",
		     "arn:aws:s3:::oracle-cdc-bucket/*"
		   ]
		 },
		 {
		   "Sid": "GetBucketLocation",
		   "Effect": "Allow",
		   "Action": "s3:GetBucketLocation",
		   "Resource": "arn:aws:s3:::oracle-cdc-bucket"
		 }
		]
		```
	- CloudWatch Logs 사용하러면 아래 권한 추가
	```json
	    {
	      "Sid": "AllowCloudWatchLogging",
	      "Effect": "Allow",
	      "Action": [
	        "logs:CreateLogGroup",
	        "logs:CreateLogStream",
	        "logs:PutLogEvents"
	      ],
	      "Resource": [
	        "arn:aws:logs:*:*:log-group:/aws/dms/*"
	      ]
	    }
	```

### 3-2 역할 생성
- IAM 콘솔
- 역할 생성
	- aws 서비스
	- 사용 사례 : DMS

	- 권한
		- DMS-S3-MinimalPolicy

	- 역할 이름 : dms-s3-role
	- 신뢰 관계
		```json
		{
		  "Version": "2012-10-17",
		  "Statement": [
		    {
		      "Effect": "Allow",
		      "Principal": {
		        "Service": "dms.amazonaws.com"
		      },
		      "Action": "sts:AssumeRole"
		    }
		  ]
		}
		```
	- 생성


## 4) DMS replication instance 생성
- DMS 콘솔
- replication instance 생성
	- 이름 : oracle-to-s3-replication
	- 인스턴스 클래스 : 선택
	- 스토리지 : 선택
	- VPC : VPN이 연결된 VPC 선택
	- 퍼블릭 엑세스 : 끄기 (사설망만 사용)
	- 나머지 기본값
	- 생성


## 5) Oracle Endpoint 생성
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


## 6) S3 Endpoint 생성
- DMS 콘솔
- Endpoint 생성
	- endpoint type : target
	- 식별자 : st-target-endpoint
	- 소스 엔진 : Amazon S3
	- Amazon S3 bucket : 버킷 선택
	- IAM role for S3 bucket : Use ana existing IAM role 선택
	- 위에서 만들었던 IAM role 선택

## 7) Migration Task 생성
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
		- 테스크 로그 - 필요시(비용 발생)
	- table mappings
		- 새 선택 규칙 추가
			- 스키마 입력 클릭
			- schema name : % (전체 선택 의미)
			- table name : % (전체 선택 의미, 다른 것들은 [참고](https://docs.aws.amazon.com/ko_kr/dms/latest/userguide/CHAP_Tasks.CustomizingTasks.TableMapping.SelectionTransformation.Wildcards.html))
			- include
	- 마이그레이션 전 평가
		- 결과 버킷 위치 설정
	- 생성
## 8) 테스트

- 오라클에서 commit 시 dms cdc 로그가 찍히는 지 
- dms 콘솔
	- dms task 상태가 `running` 이여야 함
	- cloudWatch Logs 확인(위에서 켰다면)
- S3 버킷 변경 파일 확인


