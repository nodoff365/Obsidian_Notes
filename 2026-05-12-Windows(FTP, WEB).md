```bash
#w2k22-ad DHCP, DNS 구성
Install-WindowsFeature -Name DHCP -IncludeManagementTools

도구 - DHCP - IPv4(새 범위) - ~~
관리 - 역할 및 기능 추가 - ~~

dns, 웹서버(IIS)
역할 서비스 (IP및도메인제한, windows인증, 기본인증, 로깅도구, 동적콘텐츠압축, http리디렉션, 관리서비스)

도구 - dns - 정방향 - 주영역 - 영역이름(sgm.local) - 
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
compmgmt.msc
services.msc
```


```html
<html>
<body>
<h1>SGM-WEB-2</h1>
</body>
</html>
```
