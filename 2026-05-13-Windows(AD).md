## AD 설치 및 구성
---
Windows Server(AD를 통한 인증, Group Policy를 통한 관리)
1. Standalone
	1.1 독립 실행형. 다른 서버와 상관 관계가 없음
2. Member Server(Domain Logon, Local Logon)
	2.1 DC의 자원(User, Group, Computer)을 갖어다 쓰기 위해서 구성
	2.2 Application은 Member Server에 설치를 권장함
3. Domain Controller(Domain Logon)
	3.1 Domain Controller
		3.1.1 일정한 영역
		3.1.2 Domain Name
		3.1.3 고객과 직원 계정 및 패스워드 정보 저장
		3.1.4 Application 설치 비권장 (보안 및 안정성 문제)
---
## Administrator 계정의 종류

### 1. 로컬 Administrator (= pc이름\administrator)

- **각 컴퓨터마다 독립적으로 존재**하는 계정
- AD와 **완전히 무관** — 도메인이 죽어도 사용 가능
- SAM 데이터베이스(레지스트리)에 저장
- 예: `MYPC\Administrator`, `DC01\Administrator`

### 2. 도메인 Administrator (= 도메인명\administrator)

- **AD 도메인 전체를 관리**하는 계정
- AD 데이터베이스(ntds.dit)에 저장
- 예: `CORP\Administrator`

### 3. DSRM Administrator (디렉터리 서비스 복구 모드)

- **DC 설치 시 따로 설정**하는 별도 비밀번호
- DC의 로컬 Administrator처럼 동작하지만, **일반 부팅 시에는 사용 불가**
- AD가 망가졌을 때 복구용으로만 사용
---
```bash
# w2k22-ad

서버 역할(ActiveDirectory도메인서비스)
이 서버를 도메인 컨트롤러로 승격
새 포리스트를 추가합니다 - 루트 도메인 이름 sgm.local 다음 - 포리스트기능수준 2016/DSRM암호 It12345! 다음 - DNS옵션(다음) - 추가옵션(다음) - 경로(다음) - 검토옵션(다음) - 필수구성요소확인(설치)

ncpa.cpl - ipv6 - 자동으로 DNS서버주소받기로 변경

시스템 구성(msconfig) - 부팅 - 안전부팅(Active Directory복구)
ad 복원모드 로그인 할 때 pc이름\administrator(로컬계정) 사용

대시보드 - 도구(ActiveDirectory사용자및컴퓨터) - Users(새로만들기-사용자) - 전체이름(Test_A),사용자로그온이름(a)

```

```bash
# w2k22-mem1
서버 역할(ActiveDirectory도메인서비스)
이 서버를 도메인 컨트롤러로 승격 - 배포구성(기존 도메인에 도메인~) - 도메인(선택) - sgm\administrator / It1 - 도메인컨트롤러옵션(DSRM암호 It12345!) 다음 - DNS옵션(다음) - 추가옵션(다음) - 경로(다음) - 검토옵션(다음) - 필수구성요소확인(설치)
```

```bash
# w2k22-mem2

```
