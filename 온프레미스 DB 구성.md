Rocky Linux에 MySQL 설치하는 방법이에요.

**1. MySQL 설치**
```bash
dnf install -y mysql-server
systemctl enable --now mysqld
```

**2. 초기 보안 설정**
```bash
mysql_secure_installation
```

비밀번호 설정하라고 나오면 `It12345@` 입력하세요.

**3. WordPress용 DB/유저 생성**



```bash
mysql -u root -pIt12345@
```



```sql
CREATE DATABASE wordpress;
CREATE USER 'ijo'@'%' IDENTIFIED BY 'It12345@';
GRANT ALL PRIVILEGES ON wordpress.* TO 'ijo'@'%';
FLUSH PRIVILEGES;
EXIT;
```

**4. 외부 접속 허용 (방화벽)**



```bash
firewall-cmd --permanent --add-port=3306/tcp
firewall-cmd --reload
```

**5. MySQL 외부 접속 허용 확인**



```bash
# bind-address 확인
grep bind-address /etc/my.cnf.d/*.cnf
```

`bind-address = 127.0.0.1` 이면 외부 접속 안 돼요. 그러면 아래 추가해주세요.



```bash
echo "bind-address = 0.0.0.0" >> /etc/my.cnf.d/mysql-server.cnf
systemctl restart mysqld
```

**6. 설치 확인**


```bash
mysql -u ijo -pIt12345@ -e "show databases;"
```