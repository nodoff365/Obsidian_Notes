# MySQL Backup
---
>데이터베이스 장애, 사용자 실수, 보안 사고, 시스템 이전 등에 대비하여 DB의 데이터와 구조를 안전한 저장소에 보관하고, 필요 시 원래 상태로 복구할 수 있도록 하는 작업
>[실습 자료1](https://loop.cloud.microsoft/p/eyJ1IjoiaHR0cHM6Ly9ob21lLm1pY3Jvc29mdHBlcnNvbmFsY29udGVudC5jb20vY29udGVudHN0b3JhZ2UvQ1NQX2FjNDBlMDRjLWM1ZWItNGE1Ni05ZDg0LTk0MWIyYjU4YTA1Mz9uYXY9Y3owbE1rWmpiMjUwWlc1MGMzUnZjbUZuWlNVeVJrTlRVRjloWXpRd1pUQTBZeTFqTldWaUxUUmhOVFl0T1dRNE5DMDVOREZpTW1JMU9HRXdOVE1tWkQxaUpUSXhWRTlDUVhKUGRrWldhM0ZrYUVwUllrc3hhV2RWZUZkUlV6WldjVVZHUmsxdGJYQklPRjlOVGpGSmNGaHNWRjlYYWpSclJWSTFNQzFqUm1aMlVsQndTaVptUFRBeFR6VlNXbEJNVDBnMFNsQkpVMDQyUTBKT1FqTldURXBMU0ZGSE1rZzFVRWNtWXowbE1rWW1ZVDFNYjI5d1FYQndKbkE5SlRRd1pteDFhV1I0SlRKR2JHOXZjQzF3WVdkbExXTnZiblJoYVc1bGNnPT0ifQ%3D%3D)
##### 백업이 필요한 이유
- `DROP / DELETE / TRUNCATE` 실수 
- 디스크 장애, 서버 장애
- 랜섬웨어·침해 사고
- 시스템 이전(Migration)
- 테스트/실습 환경 복제

백업의 완성은 “복구 테스트”까지 포함

---
### MySQL Backup 종류
1. 백업 방식 기준

|  구분   |     설명     |    대표 도구    |     특징      |
| :---: | :--------: | :---------: | :---------: |
| 논리 백업 | SQL 문으로 저장 | `mysqldump` | 교육·시험·일반 실무 |
| 물리 백업 | 데이터 파일 단위  | XtraBackup  |  대용량·운영 DB  |

2. 백업 범위 기준

|구분|설명|
|---|---|
|전체 백업|DB 전체(구조+데이터)|
|부분 백업|특정 테이블/DB|
|스키마 백업|테이블 구조만|
|데이터 백업|데이터만|
3. 실행 방식 기준

|구분|설명|
|---|---|
|수동 백업|명령어 직접 실행|
|자동 백업|cron 스케줄링|
# DB 백업
---
### 전체 DB 백업
1.CREATE DATABASE 포함 방식
```bash
mysqldump -u [사용자명] -p --databases [데이터베이스명] > [백업파일명.sql]

mysqldump -u root -p --databases student > student_full-db.sql
# CREATE DATABASE + USE 포함
# DB 전체 백업
```

2.CREATE DATABASE 미포함 방식
```bash
mysqldump -u [사용자명] -p [데이터베이스명] > [백업파일명.sql]

mysqldump -u root -p student > student_full.sql
# CREATE DATABASE 미포함
# DB 전체 테이블 백업
```

### DB의 특정 테이블 백업
```bash
mysqldump -u [사용자명] -p [데이터베이스명] [테이블명] > [백업파일명.sql]

mysqldump -u root -p student sungjuk > student_sungjuk.sql
# 특정 테이블만 백업
# CREATE DATABASE 미포함
```
---
# DB 복구
### 전체 DB 복구
1.CREATE DATABASE 포함 덤프 복구
```bash
mysql -u [사용자명] -p < [백업파일명.sql]

mysql -u root -p < student_full-db.sql
# DB 없어도 복구 가능
# CREATE DATABASE 포함
```

2.CREATE DATABASE 미포함 덤프 복구
```bash
mysql -u [사용자명] -p [데이터베이스명] < [백업파일명.sql]

mysql -u root -p student < student_full.sql
# DB는 미리 존재해야 함
# DB 전체 테이블 복구
```

### 특정 테이블 복구
```bash
mysql -u [사용자명] -p [데이터베이스명] < [백업파일명.sql]

mysql -u root -p student < student_sungjuk.sql
# DB는 미리 존재해야 함
# 해당 테이블만 복구
```

---
### 원격 DB 서버 백업
```bash
mysqldump -h [호스트IP] -u [사용자명] -p [데이터베이스명] > [백업파일명.sql]

mysqldump -h 192.168.200.22 -u webapp -p student > student_remote.sql
mysqldump -h 192.168.200.22 -u webapp -p --databases student --single-transaction --no-tablespaces > student_remote.sql
```

```bash
#사전 준비 sql 서버에서 권한 설정해주기
show grants for 'webapp'@'192.168.200.21';
grant select on seudent.* to 'webapp'@'192.168.200.21';
flush privileges;
```
---
### 압축 백업
### 자동 백업 스크립트 실습
```bash
#!/bin/bash  
DB=student
BACKUP_DIR=/backup/mysql
DATE=$(date +%F)
  
mkdir -p $BACKUP_DIR
  
mysqldump -u root -p 'Tj0tlf.0505' $DB \
| gzip > $BACKUP_DIR/${DB}_${DATE}.sql.gz
  
# 7일 초과 백업 삭제
find $BACKUP_DIR -type f -mtime +7 -delete
```

### 무결성 검증

---
# MySQL Trigger
---
>Trigger(트리거)는 특정 테이블에서 INSERT / UPDATE / DELETE 같은 이벤트가 발생했을 때 자동으로 실행되는 SQL 문



db생성
트리거생성
확인
##### 동작테스트
```sql
INSERT INTO users(username, email) VALUES ('nova','nova@kcci.edu');
UPDATE users SET email='nova2@kcci.edu' WHERE username='nova';
DELETE FROM users WHERE username='nova';

SELECT * FROM user_audit ORDER BY audit_id;
```

### student DB에서 sungjuk(성적), jusorok(주소록) 테이블을 대상으로 INSERT/UPDATE/DELETE 트리거 실습

##### 전체 실습 시나리오 요약
- `sungjuk`
   - INSERT/UPDATE/DELETE 시 감사로그 기록 
   - UPDATE/DELETE 시 순위 테이블(`sungjuk_ranking`) 자동 갱신/동기화 
   - DELETE는 “soft policy”로 100번 이하는 삭제 금지(정책 트리거 예시)   
- `jusorok`
   - INSERT/UPDATE/DELETE 시 감사로그 기록 
   - 전화번호/이메일 정규화(공백 제거, 소문자 처리) (BEFORE 트리거)



---
# KALI
cookpit

nikto -h https://web.kcci.edu

sqlmap
nmap