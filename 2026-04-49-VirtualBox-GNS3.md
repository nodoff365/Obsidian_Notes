## 1. GNS3
```text
# Gns3 환경 설정

Edit - Preferences - Console applications(Edit) - Xshell 5 선택 - 경로
(C:\Program Files (x86)\NetSarang\Xshell 8\xshell.exe)로 변경

Preferences - Server - Host binding(127.0.0.1)

Dynamips - IOS routers - New - New Image(IOS image) Next - Next - Next - Next - wic 0:(WIC -2T) Next - Idle-PC finder(Finish) - 우하단 Apply
```

```text
# Gns3에 VirtualBox 연결
Edit - Preferences - VirtualBox VMs(New) - Rocky9-1(Finish) - Apply(OK) 
# 같은 방법으로 9-2 추가
```

```text
# Gns3 라우터 설정
R1 

do sh ip route
네트워크 개수 라우팅테이블 개수 맞추기
```

## 2. Oracle VirtualBox
```text
# Oracle VirtualBox 환경 설정
파일 - 환경 설정 - Basic - 일반(Default Machine Folder) 저장 경로 지정
Expert - 가상 머신 - Host Key Combo(Ctrl + Alt)
```

```text
# Oracle VirtualBox 구성
머신 - 새로 만들기 - VM Name(Rocky9-1) - ISO Image - Proceed with Unattended Installation(체크 해제) 다음 - Base Memory(2048)/CPU(1)/Disk Size(50) 다음 - 완료

Rocky9-1(설정) - Expert(네트워크) - 어댑터 1 Attached to (연결되지 않음) 확인 - 시작

# 리눅스 설치
한글/설치 목적지 완료/KDUMP 비활성화/root pw It1, root가 비밀번호~~ 체크

# Rocky9-1 복제
Rocky9-1 - 복제 - 이름(Rocky9-2) - 완전한 복제 - MAC 주소 정책(NAT 네트워크 어댑터 MAC 주소만 포함) 완료
```

```text
# win10
sysdm.cpl - 컴퓨터 이름(w10)
장치 - 게스트 확장 CD 이미지 삽입 - VBoxWindowsAdditions-amd64
control - 방화벽 - 고급설정 - 에코요청 사용함
control - 전원옵션
services.msc - windows update 사용안함
ncpa.cpl 3.0.0.2/24 3.0.0.254
168.126.63.1 8.8.8.8
```

```text
# winsvr
ip 1.0.0.2/24 1.0.0.254
168.126.63.1 8.8.8.8
```