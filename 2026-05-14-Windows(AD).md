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
`cmd - netdom query fsmo`
## 1.
```bash
# w2k22-ad Active Directory 사이트 및 서비스
Sites(새사이트) - 이름(jp), 링크 확인
Subnets(새서브넷) - 접두사(20.0.0.0/24) - 개체(jp) 선택


# 스키마 마스터 전송
cmd - regsvr32 schmmgmt.dll
실행(mmc) - 파일(스냅인추가) - ad도메인사용자사이트스키마
추가 확인 - 파일다른이름으로저장바탕화면에AD로추가 - ad스키마(스키마마스터변경-변경) 닫기


# cli로 변경 
ntdsutil 
roles 
Connections
connect to server w2k22-ad.sgm.local
quit
fsmo maintance: Seize schema master
fsmo maintance: Transfer infrastructure master # 연결된 서버를 인프라 마스터로 만듭니다.
fsmo maintance: Transfer naming master        # 연결된 서버를 명명 마스터로 만듭니다.
fsmo maintance: Transfer PDC                  # 연결된 서버를 PDC로 만듭니다.
fsmo maintance: Transfer RID master           # 연결된 서버를 RID 마스터로 만듭니다.
fsmo maintance: Transfer schema master        # 연결된 서버를 스키마 마스터로 만듭니다.
```

```bash
# w2k22-mem1
ad 사용자 및 컴퓨터 모든작업 - 작업마스터(RID/PDC/인프라 변경)

```