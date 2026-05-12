```bash
# powershell에서 설치도 가능 
Install-WindowsFeature -Name DHCP -IncludeManagementTools
```

```bash
# w2k22-mem1 FTP 구성
역할 서비스 (IP및도메인제한, windows인증, 기본인증, 로깅도구, 동적콘텐츠압축, http리디렉션, ftp서버, 관리서비스)

# C 드라이브에서 아래와 같은 구조로 생성
01_FTP/
├── a/
│   └── a.txt
├── b/
│   └── b.txt
├── LocalUser/
│   ├── a/
│   │   └── a1.txt
│   └── b/
│       └── b1.txt
└── All.txt
```

```bash
compmgmt.msc # 컴퓨터 관리, 계정 추가
services.msc # 서비스
```

```bash
# WEB SITE 구축
C - web 폴더 생성 - index.html 작성
# index.html 내용
<html>
<body>
<h1>SGM-MAIN-1</h1>
</body>
</html>

IIS 관리자 - 사이트 - 웹사이트 추가 - 사이트이름/실제경로 지정(경로 C:\web)
```

![[Pasted image 20260512142916.png]]

```bash
# w2k22-ad
서버관리자 - 관리 - 역할 및 기능 추가 - 다음 - 다음 - 다음 - DHCP 서버/DNS 서버/웹 서버(IIS) 다음 - 

DHCP - IPv4(새 범위) - 다음 - 이름(sgm_dhcp) - 
```

```bash
# w2k22-mem1

IIS관리자
ftp 방화벽인바운드 추가
서비스 Microsoft FTP Service 재시작
```

```bash
# w2k22-mem2


```



https://learn.microsoft.com/en-us/sysinternals/downloads/
Sysinternsal Suite - Bginfo64