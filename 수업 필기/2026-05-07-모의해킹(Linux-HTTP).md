```bash
dnf install -y httpd # apache 웹서버
#dnf install -y nginx

#설정 파일
vi /etc/httpd/conf/httpd.conf
91 ServerAdmin web@gmseo.local #관리자 계정명 변경
149 Indexes 삭제 #디렉토리 리스닝 기능

vi /etc/httpd/conf.d/welcome.conf
mv /etc/httpd/conf.d/{welcome.conf,welcome.conf.bak}
mkdir /var/www/html/{a,b,c,d,e,f,g}
vi /var/www/html/index.html
<html>
<body>
<h1>MAIN-GMSEO-WEB-1</h1>
</body>
</html>

systemctl enable --now httpd
firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

```bash
# 9-1
mkdir /var/www/blog
vi /var/www/blog/index.html
<html>
<body>
<h1>BLOG-GMSEO-WEB-1</h1>
</body>
</html>

vi /etc/httpd/conf/httpd.conf
NameVirtualHost *:80

<VirtualHost *:80>
        ServerName      blog.gmseo.local
        DocumentRoot    /var/www/blog
</VirtualHost>

<VirtualHost *:80>
        ServerName      www.gmseo.local
        ServerAlias     gmseo.local
        DocumentRoot    /var/www/blog
</VirtualHost>

# 9-2
vi /etc/httpd/conf/httpd.conf
NameVirtualHost *:80

<VirtualHost *:80>
         ServerName      blog.gmseo.local
         DocumentRoot    /var/www/blog
 </VirtualHost>
 
 <VirtualHost *:80>
         ServerName      blog.gmseo.local
         DocumentRoot    /var/www/intra
 </VirtualHost>
    
<VirtualHost *:80>
	ServerName      www.gmseo.local
	ServerAlias     gmseo.local
	DocumentRoot    /var/www/blog
</VirtualHost>
```

```bash
dnf autoremove -y httpd
rm -rf /etc/httpd/ /var/www
firewall-cmd --permanent --remove-port=80/tcp
firewall-cmd --reload
```

```bash
#dns
rm -rf /var/named/{1,2}
systemctl restart named
ls -al /var/named/
```

```bash
# 9-1
dnf install -y httpd

mkdir /var/www/blog
vi /var/www/html/index.html
<html>
<body>
<h1>MAIN-GMSEO-WEB-1</h1>
</body>
</html>

vi /var/www/blog/index.html
<html>
<body>
<h1>BLOG-GMSEO-WEB-1</h1>
</body>
</html>

vi /etc/httpd/conf.d/vir.conf
NameVirtualHost *:80

<VirtualHost *:80>
        ServerName      www.gmseo.local
        ServerAlias     gmseo.local
        DocumentRoot    /var/www/html
</VirtualHost>

<VirtualHost *:80>
        ServerName      blog.gmseo.local
        DocumentRoot    /var/www/blog
</VirtualHost>

mv /etc/httpd/conf.d/{welcome.conf,welcome.conf.bak}

vi /var/named/1
blog	A	10.0.0.11 # 추가
systemctl restart named

systemctl enable --now httpd
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload
--------------------------------------------------------
# 9-2
mkdir /var/www/{blog,intra}
vi /var/www/html/index.html
<html>
<body>
<h1>MAIN-GMSEO-WEB-2</h1>
</body>
</html>

vi /var/www/blog/index.html
<html>
<body>
<h1>BLOG-GMSEO-WEB-2</h1>
</body>
</html>

vi /var/www/intra/index.html
<html>
<body>
<h1>INTRA-GMSEO-WEB-2</h1>
</body>
</html>

vi /etc/httpd/conf.d/vir.conf
NameVirtualHost *:80

<VirtualHost *:80>
        ServerName      www.gmseo.local
        ServerAlias     gmseo.local
        DocumentRoot    /var/www/html
</VirtualHost>

<VirtualHost *:80>
        ServerName      blog.gmseo.local
        DocumentRoot    /var/www/blog
</VirtualHost>

<VirtualHost *:80>
        ServerName      intra.gmseo.local
        DocumentRoot    /var/www/intra
</VirtualHost>

mv /etc/httpd/conf.d/{welcome.conf,welcome.conf.bak}

# 9-1에서
vi /var/named/1
blog    A       10.0.0.12
intra   A       10.0.0.12
systemctl restart named

systemctl enable --now httpd
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload
--------------------------------------------------------
# 9-3
mkdir /var/www/intra

vi /var/www/html/index.html
<html>
<body>
<h1>MAIN-GMSEO-WEB-3</h1>
</body>
</html>

vi /var/www/intra/index.html
<html>
<body>
<h1>INTRA-GMSEO-WEB-3</h1>
</body>
</html>

vi /etc/httpd/conf.d/vir.conf
NameVirtualHost *:80

<VirtualHost *:80>
        ServerName      www.gmseo.local
        ServerAlias     gmseo.local
        DocumentRoot    /var/www/html
</VirtualHost>

<VirtualHost *:80>
        ServerName      intra.gmseo.local
        DocumentRoot    /var/www/intra
</VitrualHost>

mv /etc/httpd/conf.d/{welcome.conf,welcome.conf.bak}

# 9-1에서
vi /var/named/1
intra   A       10.0.0.13
systemctl restart named

systemctl enable --now httpd
firewall-cmd --permanent --add-port=80/tcp
firewall-cmd --reload
```

|경로|설명|
|---|---|
|`/etc/httpd/conf/httpd.conf`|메인 설정 파일, 직접 여기에 VirtualHost 작성|
|`/etc/httpd/conf.d/*.conf`|별도 파일로 분리, httpd.conf가 자동으로 불러옴|

결과는 동일합니다. `conf.d/`에 따로 파일 만드는 게 **관리하기 더 편해서** 실무에서 주로 사용합니다.


## 2.ACL
```bash
# rocky 9-1
# blog 페이지 ip 접근제어 설정/ACL
vi /etc/httpd/conf.d/vir.conf
<Directory "/var/www/blog">
        Order   deny,allow #뒤에 있는게 우선 실행
        Allow from 10.0.0.101
        Deny from all
</Directory>

systemctl restart httpd
------------------------------------------------------
# rocky 9-2
vi /etc/httpd/conf.d/vir.conf
<Directory "/var/www/blog">
        Order   deny,allow #뒤에 있는게 우선 실행
        Allow from 10.0.0.101
        Deny from all
</Directory>

systemctl restart httpd

# blog.gmseo.local / win10에서만 접속가능 11에선 안됨
```

```bash
# rocky 9-2~3
# intra 페이지 계정 접근제어 설정/ACL
vi /etc/httpd/conf.d/vir.conf
<Directory "/var/www/intra">
        AllowOverride AuthConfig
</Directory>

vi /var/www/intra/.htaccess
AuthName        "Auth Test"
AuthType        Basic
AuthUserFile    /web/.user
Require user    a b

mkdir /web
htpasswd -c /web/.user a #제일 처음 계정 만들 때 -c 넣고 추후엔 X
htpasswd /web/.user b
systemctl restart httpd

#intra.gmseo.local에서 테스트
```

```bash
nslookup
dig @8.8.8.8 naver.com +trace
```


```
<html>
<body>
<h1>MAIN-SGM-WEB-2</h1>
</body>
</html>
```