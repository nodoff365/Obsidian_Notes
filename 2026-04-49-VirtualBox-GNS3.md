```shell
# Rocky9-1
dnf install -y dhcp-server
vi /etc/dhcp/dhcpd.conf # 환경 설정 파일 수정
:$r /usr/share/doc/dhcp-server/dhcpd.conf.example
# 문서 맨 끝에 첨부
:1,51d (enter) # 1~51번재 줄 삭제
:10,28d # 10~28번째 줄 삭제
:14,$d # 14~마지막 줄 삭제

vi /var/lib/dhcpd/dhcpd.leases #dhcp ip 임대목록 파일
```


```shell
# Rocky9-2
dnf install -y vsftpd
reload
:!bash
```