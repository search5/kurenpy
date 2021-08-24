# 데이터 서버 이전 작업 결과서

## 준비
1. 이전 대상 서버
1. 신규 대상 서버

## 작업 진행

### 신규 대상 서버에 운영체제 설치

Debian 11 Bullseye

### 신규 대상 서버에 패키지 설치

시스템 패키지 설치
```
# usermod -aG sudo jiho
# apt install net-tools vim build-essential ntpdate
```

시스템 시간 조정
```
$ sudo timedatectl set-timezone Asia/Seoul
$ sudo ntpdate time2.kriss.re.kr
```

OTP 사용 설정

<https://www.lesstif.com/system-admin/google-authenticator-linux-ssh-otp-24444948.html> 참고

PHP 설치
```
$ sudo apt install php7.4 php-common php-gd php-igbinary php-imagick php-intl php-mbstring
$ sudo apt install php-pgsql php-redis php-zip php-bcmath php-gmp php-json php-ldap php-xml
$ sudo apt install php-readline php-opcache php-fpm libmagickcore-6.q16-6-extra
```

### 아파치 설치
```
$ sudo apt install apache2
```

### PostgreSQL 설치
```
$ sudo apt install postgresql-13
```

### REDIS 설치
```
$ sudo apt install redis-server
```

### PostgreSQL 사용자 및 데이터베이스 생성
```
$ sudo -u postgres -s /bin/bash
$ psql
postgres=# create role insignal with login PASSWORD 'xDIa3x8R7?xc';
postgres=# create database owncloud encoding 'utf-8' owner insignal;
postgres=# \q
```

### 이전 대상 서버로부터 파일 복사하기

단, 신규 서버의 ssh 공개키가 이전 서버의 root의 ssh 설정에 추가되어 있어야 한다.

```
# mkdir /var/www/owncloud
# cd /var/www/owncloud
# rsync -Aavxt root@data.insignal.co.kr:/var/www/owncloud .
```

### 이전 대상 서버에서 DB 백업받기
```
$ PGPASSWORD="xDIa3x8R7?xc" pg_dump owncloud -h 127.0.0.1 -U insignal -f nextcloud-sqlbkp_`date +"%Y%m%d"`.bak
```

### 신규 서버에서 DB 데이터 이전하기
```
$ PGPASSWORD="xDIa3x8R7?xc" psql -h 127.0.0.1 -U insignal -d owncloud -f nextcloud-sqlbkp.bak
```

### DDNS 파일 추가
DDNS 파일은 서버의 IP가 바뀌었을때 DNS 서버에 변경된 IP를 통지하는 프로그램이다.

```
$ sudo vi /etc/systemd/system/ddns.service
[Unit]
Description=Dynamic DNS Update
After=networking.service

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/sbin/ddns.sh

[Install]
WantedBy=multi-user.target
```

```
$ sudo vi /sbin/ddns.sh
#!/bin/sh

wget -O - --http-user=embryo --http-passwd=s3lb4ecw0h 'http://dyna.dnsever.com/update.php?host[data.insignal.co.kr]'
$ sudo chmod +x /sbin/ddns.sh
```

### 파이썬 가상환경 및 인증 모듈 설치

```
$ sudo apt install python3-pip python3-venv certbot python3-certbot-apache
```

### 아파치 설정 파일 수정

```
$ sudo vi/etc/apache2/sites-enabled/000-default.conf 
<VirtualHost *:80>
        ServerName data.insignal.co.kr

        ServerAdmin search5@insignal.co.kr
        DocumentRoot /var/www/owncloud

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <Directory /var/www/owncloud/>
            Options +FollowSymlinks
            AllowOverride All

            <IfModule mod_dav.c>
                Dav off
            </IfModule>

            SetEnv HOME /var/www/owncloud
            SetEnv HTTP_HOME /var/www/owncloud
        </Directory>
</VirtualHost>
```

### 인증서 장착
```
$ sudo certbot --apache -d data.insignal.co.kr
$ sudo systemctl restart apache2
```

### 아파치 설정 업데이트(반드시 CERTBOT 이후 실행)
```
$ vi /etc/apache2/sites-enabled/000-default-le-ssl.conf
...
    <IfModule mod_headers.c>
      Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
    </IfModule>
...
```

### 아파치 설정 및 모듈 추가
```
$ sudo a2enconf php7.4-fpm
$ sudo a2enmod headers proxy proxy_fcgi rewrite ssl
$ sudo systemctl restart apache2
```

### PHP 설정 업데이트
```
$ sudo vi /etc/php/7.4/fpm/php.ini
memory_limit = 512M
$ sudo systemctl restart php7.4-fpm
```

### www-data Crontab 설정(단, www-data 계정으로 실행해야 함)

```
$ crontab -e
*/5  *  *  *  * php -f /var/www/owncloud/cron.php
```
