## MAIL 구성
```shell
# Rocky9-3
vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 rocky9-3
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6 rocky9-3
10.0.0.13       rocky9-3

dnf install -y sendmail sendmail-cf dovecot

useradd x
useradd y
echo "IT1" | passwd --stdin x # y도

/etc/mail #설정파일
vi /etc/mail/sendmail.mc
21 #자세히는 9 -> 14 로 변경
56 dnl 삭제
57 dnl 삭제
121 DAEMON_OPTIONS(`Port=smtp, Name=MTA')dnl # Addr 부분 전체 삭제

m4 /etc/mail/sendmail.mc > /etc/mail/sendmail.cf

vi /etc/mail/local-host-names
mx1.sgm.local # 추가
sgm.local
rocky9-3

vi /etc/mail/access
Connect:10.0.0.0/255.255.255.0          RELAY
Connect:10.0.0                          RELAY   
Connect:mx1.sgm.local                   RELAY
Connect:sgm.local                       RELAY
Connect:rocky9-3                        RELAY

makemap hash /etc/mail/access < /etc/mail/access

vi /etc/group
mail:x:12:x,y # mail에 x,y 등록

#받는 메일
vi /etc/dovecot/dovecot.conf
24 #주석제거
30 #주석제거

vi /etc/dovecot/conf.d/10-auth.conf
10 disable_plaintext_auth = no #주석제거, yes->no

vi /etc/dovecot/conf.d/10-mail.conf
25 #주석제거

vi /etc/dovecot/conf.d/10-master.conf
19 #주석제거
40 #주석제거

vi /etc/dovecot/conf.d/10-ssl.conf
8 ssl = no #required -> no

systemctl enable --now sendmail
systemctl enable --now dovecot
ss -nat #25,110,143 포트 확인

firewall-cmd --permanent --add-port={25,110,143}/tcp
firewall-cmd --reload
```


```bash
#win10 Thunderbird
계정 허브
새로운 계정 허브에서 계정 생성(C) #체크해제

설정 - 계정 설정 - 새 계정(메일 계정) - 이름(w10)/이메일 주소(x@sgm.local) - manual configuration - 
호스트명 mx1.sgm.local 
연결보안 없음
인증방법 자동감지

```

```bash
# 실습 후 삭제
dnf autoremove -y sendmail sendmail-cf dovecot
rm -rf /etc/mail /etc/dovecot

userdel -r x
userdel -r y

firewall-cmd --permanent --remove-port={25,110,143}/tcp
firewall-cmd --reload
```
