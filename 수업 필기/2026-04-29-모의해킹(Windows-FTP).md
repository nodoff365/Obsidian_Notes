## mem1
```text
w2k22-mem1

ftp active mode

c드라이브-01_ftp 파일 생성-쉬프트우클릭파워쉘열기
fsutil file createnew ftp.txt 1024 - 

실행 compmgmt.msc(컴퓨터 관리)-로컬사용자및그룹-사용자(새사용자)-사용자이름,암호, 다음로그온~~(해제) 사용자가~,암호사용기간 체크

서버 관리자(대시보드) - 관리(역할및기능추가) - 다음 - 역할서비스 웹서버해제 /ftp서버 체크


도구 - IIS관리자 - 확장후 기존사이트삭제 - ftp사이트생성

ftp 권한부여규칙 - 허용규칙추가 - 지정한 사용자 a,b 읽기쓰기 체크

ftp 인증 - 기본인증,익명 인증 사용

ftp 디렉터리 검색 unix

ftp 메시지
메시지에서 사용자 변수 지원체크 
배너 시작 %username% 님! 안녕하세요!
	배너 끝내기 %username% 님! 몰더및파일정리철저히
	최대연결수 잠시 후 재접속해주시기 바랍니다.

ftp 방화벽 지원 
ftp 사용자 격리
ftp 요청 필터링 파일 이름 확장명(.txt 거부)
						명령(STOR 거부)
```



```text
# chroot랑 같은 기능

ftp 사용자 격리 사용자이름디렉터리
사용자격리는안되지만홈디렉토리로는 가짐 

ftp 사이트 우클릭 - 탐색 - 폴더 a,b생성 - 폴더 a에 a.txt,폴더b에 b.txt 생성 - anonymous 용 default 생성

사용자이름실제디렉터리(전역 가상 디렉터리 사용)-적용
사용자격리
localuser(파일명정해져있음) - anonymous 용 public 생성 , a,b도생성 

ftp 접속 후 dir로 경로나 생성한 txt 파일로 옵션 확인
```


```text
전역설정(사이트 밑에 있는 곳에서가 아닌 최상단)
ftp 방화벽 지원
65000-65010
10.0.0.22 (ftp 서버 ip)


전역설정서버 재부팅방법
services.msc - 
microsoft ftp service

netstat -na -p tcp
```