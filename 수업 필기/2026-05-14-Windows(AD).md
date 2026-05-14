---

---
### FSMO
특정 작업을 수행할 때 작업의 일관성을 유지하기 위해서 특정 서버가 마스터 역할을 수행하는 것

---
1.1 Forest 영역
	1.1.1 Schema master(포리스트 전체에서 스키마의 일관성 유지)
	1.1.2 Domain naming master(포리스트 전체에서 Domain Nname의 일관성 유지)

1.2 Domain 영역
	1.2.1 RID Master(사용자 계정 관리 RID는 기본적으로 DC에 500개가 할당. 그 이상의 사용자 계정 생성 시 RID Master 역할을 하는 DC에서 할당 받아야함)
	1.2.2 PDC: 사용자의 패스워드 변경 사항 추적
	서버 시간동기화(Kerberose환경에서는 서버끼리 5분이상 시간차이 발생시 인증불가) 그룹 정책 시작점
	1.2.3 Infra Structure Master:상호 참조하는

---
## 1. FSMO 역할 전송/탈취
>현재 FSMO 역할 확인 `cmd - netdom query fsmo`

### 1.1 GUI 방법
| 도구            | 대상 역할         | 경로                                                                         |
| ------------- | ------------- | -------------------------------------------------------------------------- |
| AD 사용자 및 컴퓨터  | RID, PDC, 인프라 | 도메인 우클릭 → 작업 마스터 → 각 탭에서 변경                                                |
| AD 도메인 및 트러스트 | 도메인 명명 마스터    | 도메인 우클릭 → 작업 마스터 → 변경                                                      |
| MMC (AD 스키마)  | 스키마 마스터       | `cmd - regsvr32 schmmgmt.dll` → 실행(mmc) → 스냅인 추가 → AD 스키마 우클릭 → 스키마 마스터 변경 |
### 1.2 CLI 방법
>5개 역할 전부 가능
```bash
ntdsutil
roles
connections
  connect to server <대상서버>   # 역할을 받을 서버에 접속
  quit

# 전송 (Transfer) - 기존 서버 정상일 때
Transfer schema master
Transfer naming master
Transfer PDC
Transfer RID master
Transfer infrastructure master

# 탈취 (Seize) - 기존 서버 장애 시
Seize schema master
Seize naming master
Seize PDC
Seize RID master
Seize infrastructure master
```

---
## 2. DC 강제삭제 메타데이터 삭제

```
ntdsutil
roles
connections
connect to server w2k22-ad.sgm.local
quit
transfer naming master
quit
metadata cleanup
connections
connect to server w2k22-mem1.sgm.local
quit
select operation target
list site
select site 0
list domain in iste
select domain 1
list servers for domain in site
select server 0
list naming context
select naming context 6
quit
remove selected server
remove selected naming context
remove selected domain
```



12:10~ 공유폴더실습

## 3. 

























