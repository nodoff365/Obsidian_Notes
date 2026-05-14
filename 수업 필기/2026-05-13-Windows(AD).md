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
대시보드 - 관리 - 역할및기능추가 - 시작하기전(다음) - 설치유형(다음) - 서버선택(다음) - 서버역할(ActiveDirectory도메인서비스) - 기능(다음) - AD DS(다음) - 확인(설치) - 

이서버를도메인컨트롤러로승격 - 
배포구성(새포리스트를추가합니다) - 루트 도메인 이름 (sgm.local) 다음 - 도메인컨트롤러옵션(포리스트기능수준 2016/DSRM암호 It12345!) 다음 - DNS옵션(다음) 자식도메인만위임체크 - 추가옵션(다음) - 경로(다음) - 검토옵션(다음) - 필수구성요소확인(설치)
---------------------------------------------------
ipv4 - 기본설정dns서버 10.0.0.21로 변경
ncpa.cpl - ipv6 - 자동으로 DNS서버주소받기로 변경
---------------------------------------------------
시스템 구성(msconfig) - 부팅 - 안전부팅(Active Directory복구)
# ad 복원모드 로그인 할 때 pc이름\administrator(로컬계정) 사용/pw는 DSRM암호 It12345!

#AD 계정 생성
대시보드 - 도구(ActiveDirectory사용자및컴퓨터) - Users(새로만들기-사용자) - 전체이름(Test_A),사용자로그온이름(a) - 암호(It12345!),다음~~ 체크해제,사용자~~,암호~~ 체크(다음) - 마침
```
![[Pasted image 20260514093633.png]]
```bash
# w2k22-mem1 쌍둥이 구성

서버 역할(ActiveDirectory도메인서비스)
이 서버를 도메인 컨트롤러로 승격 - 배포구성(기존 도메인에 도메인~) - 도메인(선택) - sgm\administrator / It1 - 도메인컨트롤러옵션(DSRM암호 It12345!) 다음 - DNS옵션(다음) - 추가옵션(다음) - 경로(다음) - 검토옵션(다음) - 필수구성요소확인(설치)

1차dns 10.0.0.21(도메인컨트롤러쪽으로)
```

```bash
# w2k22-mem2 멤버등록,자식

서버관리자-대시보드-관리-역할및기능추가-시작하기전(다음)-설치유형(다음)-서버선택(다음)-서버역할(AD도메인서비스)-

기존포리스트에새도메인을추가합니다-도메인유형선택(자식도메인)-선택(sgm\administrator, It12345!)-새도메인이름(busan)다음-DRSM암호It12345!-DNS옵션(다음)-추가옵션(다음)-경로(다음)-검토옵션(다음)-필수구성요소확인(설치)
-----------------------------------------------------------
# 정방향 조회 영역에 상대 DNS를 나의 보조영역에 추가
# w2k22-ad
정방향조회영역 - 새영역 - 다음 - 보조영역(다음) - 영역이름(busan.sgm.local) 다음 - 마스터서버(10.0.0.23) 다음 - 마침

sgm.local - 속성 - 영역전송 - 영역전송허용(다음 서버로만) 편집 - 10.0.0.23 추가

# w2k22-mem2
정방향조회영역 - 새영역 - 다음 - 보조영역(다음) - 영역이름(sgm.local) 다음 - 마스터서버(10.0.0.21) 다음 - 마침

busan.sgm.local - 속성 - 영역전송 - 영역전송허용(다음 서버로만) 편집 - 10.0.0.21 추가
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
대시보드-구성요소제거-AD도메인-도메인컨트롤러수준내리기
대시보드-구성요소제거-AD도메인, DNS 제거
서버관리자-로컬서버-컴퓨터이름-변경-자세히-DNS접미사 삭제

sysdm.cpl- 변경 - 작업그룹(WORKGROUP) # 도메인 멤버 가입시 
```

1. 부산지역의 AD관리자 부재 중인 경우
1.1. 부산지역의 부장이 서울 본사의 AD 관리자에게 계정 생성 요청
2. 서울지역의 AD관리자 부재 중인 경우
2.1. 서울지역의 부장이 부산 지사의 AD 관리자에게 계정 생성 요청

3. 부산 지역의 팀원이 서울로 출장 중 서울 본사에서 부산 계정으로 접속 가능한지
3.1. 가능 후 권한?
```bash
# win10 / dns 10.0.0.23
sysdm.cpl- 변경 - busan.sgm.local
```


dcdiag






1.ad 쌍둥이
2.멤버등록
3.자식등록

| 자식도메인(대규모지사)             | RODC(소규모 지사)      |
| ------------------------ | ----------------- |
| 지역제 관리자가 존재              | 지역에관리자가 부재해도됨     |
| 물리적인 보안을담보할 수 있어야함(접근통제) | 물리적인보안을담보할수없어도됨   |
| 상위단에부모도메인이존재해야함          | 상위단에쓰기가능한DC가존재해야함 |

부모자식간상호 trust관계형성
부모dc는 자식을완벽하게통제가능
자식dc는 부모dc에대한일반관리만가능

