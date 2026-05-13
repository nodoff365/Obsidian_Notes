## AD 설치 및 구성
---
Windows Server(AD를 통한 인증, Group Policy를 통한 관리)
1. Standalone
	1.1 독립 실행형. 다른 서버와 상관 관계가 없음
2. Member Server(Domain Logon, Local Logon)
	2.1 DC의 자원(User, Group, Computer)을 쓰기 위해서 구성
	2.2 Application은 Member Server에 설치를 권장함
3. Domain Controller(Domain Logon)
	3.1 Domain Controller
		3.1.1 일정한 영역
		3.1.2 Domain Name
		3.1.3 고객과 직원 계정 및 패스워드 정보 저장
		3.1.4 Application 설치 비권장 (보안 및 안정성 문제)

### Administrator 계정의 종류
---
#### 1. 로컬 Administrator
- 각 컴퓨터마다 **독립적으로 존재**
- AD(Active Directory)와 무관하게 **SAM 데이터베이스**에 저장
- 도메인이 없어져도 항상 로그인 가능
- Windows 설치 시 최초 생성되는 계정
---
#### 2. 도메인 Administrator
- AD 설치(DC 승격) 시 생성되는 계정
- **ntds.dit** 데이터베이스에 저장
- 도메인에 가입된 모든 PC에서 사용 가능
- DC(Domain Controller)가 없으면 인증 불가
---
#### 3. DSRM(Directory Services Restore Mode) Administrator
- DC 승격 시 **별도로 설정하는 비밀번호**
- AD 서비스가 손상됐을 때 복구 목적으로만 사용
- 안전 부팅(복구 모드) 상태에서만 로그인 가능
- 일반 부팅 상태에서는 사용 불가
---
```bash
# w2k22-ad

서버 역할(ActiveDirectory도메인서비스)
이 서버를 도메인 컨트롤러로 승격
새 포리스트를 추가합니다 - 루트 도메인 이름 sgm.local 다음 - 포리스트기능수준 2016/DSRM암호 It12345! 다음 - DNS옵션(다음) 자식도메인만위임체크  - 추가옵션(다음) - 경로(다음) - 검토옵션(다음) - 필수구성요소확인(설치)

ncpa.cpl - ipv6 - 자동으로 DNS서버주소받기로 변경

시스템 구성(msconfig) - 부팅 - 안전부팅(Active Directory복구)
ad 복원모드 로그인 할 때 pc이름\administrator(로컬계정) 사용

#AD 계정 생성
대시보드 - 도구(ActiveDirectory사용자및컴퓨터) - Users(새로만들기-사용자) - 전체이름(Test_A),사용자로그온이름(a) - 암호(It12345!),다음~~ 체크해제,사용자~~,암호~~ 체크(다음) - 마침
```

```bash
# w2k22-mem1
# 원격지에서 계정 사용위해 도메인 멤버로 가입
서버 역할(ActiveDirectory도메인서비스)
이 서버를 도메인 컨트롤러로 승격 - 배포구성(기존 도메인에 도메인~) - 도메인(선택) - sgm\administrator / It1 - 도메인컨트롤러옵션(DSRM암호 It12345!) 다음 - DNS옵션(다음) - 추가옵션(다음) - 경로(다음) - 검토옵션(다음) - 필수구성요소확인(설치)

구성요소제거ad도메인 도메인컨트롤러내리기
컴퓨터이름도메인변경-자세히-sgm.local 삭제

1차dns 10.0.0.21(도메인컨트롤러쪽으로)
```

```bash
# w2k22-mem2

```


```
#
# AD DS 배포용 Windows PowerShell 스크립트
#

Import-Module ADDSDeployment
Install-ADDSForest `
-CreateDnsDelegation:$false `
-DatabasePath "C:\Windows\NTDS" `
-DomainMode "WinThreshold" `
-DomainName "sgm.local" `
-DomainNetbiosName "SGM" `
-ForestMode "WinThreshold" `
-InstallDns:$true `
-LogPath "C:\Windows\NTDS" `
-NoRebootOnCompletion:$false `
-SysvolPath "C:\Windows\SYSVOL" `
-Force:$true
```

```bash
# AD, DC 제거
서버관리자-로컬서버-컴퓨터이름-변경-자세히-DNS접미사 삭제
sysdm.cpl- 변경 - 작업그룹(WORKGROUP)
```












ad 쌍둥이
멤버등록
자식등록