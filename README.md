# docker
Windows 使用 Docker Desktop &amp; Laradock 部屬本地環境

### Step1. 首先確認以下事項已完成   

1.WSL2已正確運行    
    ps.這個如果沒有可以先不理,後續會要求安裝   
    相關連結: https://docs.docker.com/desktop/windows/wsl/    
  
2.CPU為64位元 

3.有4G以上的RAM   

4.BIOS的虛擬化技術已啟用  

### Step2. 下載Docker Desktop   
https://www.docker.com/products/docker-desktop   
安裝完後重新啟動電腦     

### Step3. 取得Laradock
找個地方放上資料夾git clone Laradock  
```
git clone https://github.com/laradock/laradock.git laradock
```
### Step4. 修改設定檔
進入Laradock資料夾
並且複製已經準備好的範例檔 env.example
```
cd laradock
cp .env.example .env
```
~~將DATA_PATH_HOST=~/.laradock/data修改為DATA_PATH_HOST=./data~~   
~~(沒調整的話 mysql or mariadb 的 container 可能會無法啟動)~~

其他調整暫時先略過

### Step5. 啟動環境所需要的容器

docker-compose up -d 放入想使用的服務   
自行搭配相關容器
```
(2擇1)
 docker-compose up -d nginx mariadb workspace   
 docker-compose up -d apache2 phpmyadmin  
 docker-compose up -d nginx phpmyadmin workspace
 winpty docker exec -it laradock_workspace_1 //bin//bash

```
我先以第2項來建立,原因是比較符合我們目前的環境   
完成後,進入ppma看看是否成功
http://localhost:8081/

預設帳密
---
管理員  
伺服器: mysql  
使用者名稱: root  
密碼: root  

使用者  
伺服器: mysql  
使用者名稱: default   
密碼: secret  

### Step6. 設定vhost

1.修改windows的host 加上網址名

進入 C:\Windows\System32\drivers\etc 打開 hosts 加入
```
127.0.0.1  		project-1.test
```
之後再進入 laradock\apache2\sites 打開 default.apache.conf

將原先的保留並且複製  
修改複製的內容, 將以下內容修改為自己的專案目錄  
```
 ServerName project-1.test
 DocumentRoot /var/www/project-1
 <Directory "/var/www/project-1">
```
完整修改過後   
```
<VirtualHost *:80>
  ServerName laradock.test
  DocumentRoot /var/www
  Options Indexes FollowSymLinks

  <Directory "/var/www">
    AllowOverride All
    <IfVersion < 2.4>
      Allow from all
    </IfVersion>
    <IfVersion >= 2.4>
      Require all granted
    </IfVersion>
  </Directory>

  ErrorLog /var/log/apache2/error.log
  CustomLog /var/log/apache2/access.log combined
</VirtualHost>

<VirtualHost *:80>
  ServerName project-1.test
  DocumentRoot /var/www/project-1
  Options Indexes FollowSymLinks

  <Directory "/var/www/project-1">
    AllowOverride All
    <IfVersion < 2.4>
      Allow from all
    </IfVersion>
    <IfVersion >= 2.4>
      Require all granted
    </IfVersion>
  </Directory>

  ErrorLog /var/log/apache2/error.log
  CustomLog /var/log/apache2/access.log combined
</VirtualHost>

```
改完需要先重啟 apache的 container   
可從Docker Desktop介面重啟 或是用指令   
```
docker ps -a
docker restart laradock_apache2 or docker restart 容器編號
```
完成後,進入網址看看是否成功
http://project-1.test

### Step6. 專案mysql連線

修改專案的database設定檔  
通常只改三個地方  
```
    'hostname' => 'mysql',
    'username' => 'root',
    'password' => 'root',
```
完成後,進入網址看看是否能成功連線   
http://project-1.test

### Step7. 簡化架站流程

寫bat檔,輸入測試網站名即可馬上使用  

```
@echo off 
:starttype
@SET /P project_name=Enter your project name : 
@SET /P host_name=Enter your host name : 

@echo project_name:%project_name%
@echo host_name:%host_name%

CHOICE /C YR /M "Confirm the project name and host name you entered, or type R/r to edit."
if errorlevel 2 goto starttype
if errorlevel 1 goto typein


:typein
@echo waiting ....
@cd C:\Windows\System32\drivers\etc

@echo 127.0.0.1       wei.%host_name%.com  >>  hosts

@cd C:\docker\laradock\apache2\sites

@echo ^<VirtualHost *:80^>                              >>  default.apache.conf   
@echo   ServerName     wei.%host_name%.com              >>  default.apache.conf 
@echo   DocumentRoot /var/www/%project_name%            >>  default.apache.conf 
@echo   Options Indexes FollowSymLinks                  >>  default.apache.conf 
@echo    ^<Directory "/var/www/%project_name%"^>        >>  default.apache.conf 
@echo       AllowOverride All                           >>  default.apache.conf 
@echo      ^<IfVersion ^< 2.4 ^>                        >>  default.apache.conf 
@echo           Allow from all                          >>  default.apache.conf 
@echo      ^</IfVersion^>                               >>  default.apache.conf 
@echo      ^<IfVersion ^>^= 2.4^>                       >>  default.apache.conf 
@echo           Require all granted                     >>  default.apache.conf 
@echo      ^</IfVersion^>                               >>  default.apache.conf 
@echo    ^</Directory^>                                 >>  default.apache.conf 
@echo   ErrorLog /var/log/apache2/error_%project_name%.log             >>  default.apache.conf 
@echo   CustomLog /var/log/apache2/access_%project_name%.log combined  >>  default.apache.conf 
@echo  ^</VirtualHost^>                                 >>  default.apache.conf 
docker restart laradock_apache2_1
```
bat檔寫好後
右鍵以系統管理員身分執行
輸入你測試網站的名稱

修改記憶體佔用量
---
1按下Windows + R 鍵，輸入 %UserProfile% 並執行進入使用者資料夾

2新建檔案 .wslconfig ，然後記事本編輯

3 填入以下內容並儲存, memory為系統記憶體上限，這裡我限制最大2gb，可根據自身電腦配置設定

```
[wsl2]
memory=2GB
swap=0
localhostForwarding=true
```
4 然後啟動cmd命令提示符，輸入 wsl --shutdown 來關閉當前的子系統

php 加密安裝
---
mcrypt
```
apt-get update -y
apt-get install -y libmcrypt-dev libreadline-dev
pecl install mcrypt
```
laradock 修改mysql版本
---
參考網址:https://hoohoo.top/blog/laradock-mysql-set/

首先先停止所有運作中的 container
```
docker-compose down
```

將 laradock/.env 裡面的 MYSQL_VERSION=latest 修改為 5.7
```
MYSQL_VERSION=5.7
```
刪除 MySQL Database
//Delete mysql database
```
$ rm -rf ~/.laradock/data/mysql
```
重新建構 mysql image

//rebuild mysql image
```
$ docker-compose build mysql
//or try
$ docker-compose build --no-cache mysql
```
重新啟動 workspace container
```
$ docker-compose up -d nginx mysql phpmyadmin
```
## laradock 修改php版本
打开 laradock 根目录下的 .env 文件，查找 PHP_VERSION=，修改為要使用的版本

重新建構 php-fpm
```
docker-compose build php-fpm
docker-compose build workspace
```
重新啟動 workspace container
```
$ docker-compose up -d nginx mysql phpmyadmin
```

## 提供nginx bat檔,快速設定vhost
```
@echo off 
:starttype

@SET /P project_name=Enter your project name : 

@SET /P host_name=Enter your host name : 

@echo project_name:%project_name%
@echo host_name:%host_name%

CHOICE /C YR /M "Confirm the project name and host name you entered, or type R/r to edit."
if errorlevel 2 goto starttype
if errorlevel 1 goto typein

:typein

@echo.
@echo  ===============================
@echo  Select your project frame
@echo  ===============================
@echo.
@echo  1. Codeigniter
@echo.
@echo  2. Laravel
@echo.

CHOICE /N /C:12 /M "PICK A NUMBER (1,2)"%1
if errorlevel 2  goto Laravel
if errorlevel 1  goto Codeigniter

:Laravel
set name=public
:Codeigniter


@echo waiting ....
@cd C:\Windows\System32\drivers\etc

@echo 127.0.0.1   %host_name%.test  >>  hosts

@cd C:\Users\rock\Desktop\docker_sys\laradock\nginx\sites

type nul > %host_name%.conf

@echo server {																		>>  %host_name%.conf   

@echo 		listen 80 ;																>>  %host_name%.conf   
@echo 		listen [::]:80 ;														>>  %host_name%.conf   

@echo 		server_name %host_name%.test;											>>  %host_name%.conf   
@echo 		root /var/www/%project_name%/%name%;									>>  %host_name%.conf   
@echo 		index index.php index.html index.htm;									>>  %host_name%.conf   

@echo 		location / {															>>  %host_name%.conf   
@echo   		try_files $uri $uri/ /index.php$is_args$args;						>>  %host_name%.conf   
@echo 		}																		>>  %host_name%.conf   			

@echo 		location ~ \.php$ {														>>  %host_name%.conf   
@echo   		try_files $uri /index.php =404;										>>  %host_name%.conf   
@echo   		fastcgi_pass php-upstream;											>>  %host_name%.conf   
@echo   		fastcgi_index index.php;											>>  %host_name%.conf   
@echo   		fastcgi_buffers 16 16k;												>>  %host_name%.conf   
@echo   		fastcgi_buffer_size 32k;											>>  %host_name%.conf   
@echo   		fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;	>>  %host_name%.conf   
@echo   		#fixes timeouts														>>  %host_name%.conf   
@echo   		fastcgi_read_timeout 600;											>>  %host_name%.conf   
@echo   		include fastcgi_params;												>>  %host_name%.conf   
@echo 		}																		>>  %host_name%.conf   		

@echo 		location ~ /\.ht {														>>  %host_name%.conf   
@echo   		deny all;															>>  %host_name%.conf   
@echo 		}																		>>  %host_name%.conf   

@echo 		location /.well-known/acme-challenge/ {									>>  %host_name%.conf   
@echo   		root /var/www/letsencrypt/;											>>  %host_name%.conf   
@echo   		log_not_found off;													>>  %host_name%.conf   
@echo   	}																		>>  %host_name%.conf   
@echo  		error_log /var/log/nginx/%host_name%_error.log;							>>  %host_name%.conf   
@echo 		access_log /var/log/nginx/%host_name%_access.log;						>>  %host_name%.conf   
@echo 	}																			>>  %host_name%.conf   
	
docker restart laradock_nginx_1
```

docker 自動重啟

docker container update --restart=always $(docker ps -a -q)

### 支援多版本 php

參考文章 : https://msirius.medium.com/1-n-php-versions-and-projects-via-laradock-51938b337071


首先修改 laradock 底下 docker-compose.yml檔案

找到 PHP-FPM 複製並修改

先複製 ### PHP-FPM ############################################## 其下整段

再修改
```
### PHP-FPM-8.0 ##############################################
    php-fpm-8.0:
      build:
        context: ./php-fpm
        args:
          - CHANGE_SOURCE=${CHANGE_SOURCE}
          - BASE_IMAGE_TAG_PREFIX=${PHP_FPM_BASE_IMAGE_TAG_PREFIX}
          - LARADOCK_PHP_VERSION=8.0
          - LARADOCK_PHALCON_VERSION=${PHALCON_VERSION}
          - INSTALL_BZ2=${PHP_FPM_INSTALL_BZ2}
          - INSTALL_ENCHANT=${PHP_FPM_INSTALL_ENCHANT}
          - INSTALL_GMP=${PHP_FPM_INSTALL_GMP}
          - INSTALL_GNUPG=${PHP_FPM_INSTALL_GNUPG}
          - INSTALL_XDEBUG=${PHP_FPM_INSTALL_XDEBUG}
          - INSTALL_PCOV=${PHP_FPM_INSTALL_PCOV}
          - INSTALL_PHPDBG=${PHP_FPM_INSTALL_PHPDBG}
          - INSTALL_BLACKFIRE=${INSTALL_BLACKFIRE}
          - INSTALL_SSH2=${PHP_FPM_INSTALL_SSH2}
          - INSTALL_SOAP=${PHP_FPM_INSTALL_SOAP}
          - INSTALL_XSL=${PHP_FPM_INSTALL_XSL}
          - INSTALL_SMB=${PHP_FPM_INSTALL_SMB}
          - INSTALL_IMAP=${PHP_FPM_INSTALL_IMAP}
          - INSTALL_MONGO=${PHP_FPM_INSTALL_MONGO}
          - INSTALL_AMQP=${PHP_FPM_INSTALL_AMQP}
          - INSTALL_CASSANDRA=${PHP_FPM_INSTALL_CASSANDRA}
          - INSTALL_GEARMAN=${PHP_FPM_INSTALL_GEARMAN}
          - INSTALL_MSSQL=${PHP_FPM_INSTALL_MSSQL}
          - INSTALL_BCMATH=${PHP_FPM_INSTALL_BCMATH}
          - INSTALL_PHPREDIS=${PHP_FPM_INSTALL_PHPREDIS}
          - INSTALL_MEMCACHED=${PHP_FPM_INSTALL_MEMCACHED}
          - INSTALL_OPCACHE=${PHP_FPM_INSTALL_OPCACHE}
          - INSTALL_EXIF=${PHP_FPM_INSTALL_EXIF}
          - INSTALL_AEROSPIKE=${PHP_FPM_INSTALL_AEROSPIKE}
          - INSTALL_OCI8=${PHP_FPM_INSTALL_OCI8}
          - INSTALL_MYSQLI=${PHP_FPM_INSTALL_MYSQLI}
          - INSTALL_PGSQL=${PHP_FPM_INSTALL_PGSQL}
          - INSTALL_PG_CLIENT=${PHP_FPM_INSTALL_PG_CLIENT}
          - INSTALL_POSTGIS=${PHP_FPM_INSTALL_POSTGIS}
          - INSTALL_INTL=${PHP_FPM_INSTALL_INTL}
          - INSTALL_GHOSTSCRIPT=${PHP_FPM_INSTALL_GHOSTSCRIPT}
          - INSTALL_LDAP=${PHP_FPM_INSTALL_LDAP}
          - INSTALL_PHALCON=${PHP_FPM_INSTALL_PHALCON}
          - INSTALL_SWOOLE=${PHP_FPM_INSTALL_SWOOLE}
          - INSTALL_TAINT=${PHP_FPM_INSTALL_TAINT}
          - INSTALL_IMAGE_OPTIMIZERS=${PHP_FPM_INSTALL_IMAGE_OPTIMIZERS}
          - INSTALL_IMAGEMAGICK=${PHP_FPM_INSTALL_IMAGEMAGICK}
          - INSTALL_CALENDAR=${PHP_FPM_INSTALL_CALENDAR}
          - INSTALL_FAKETIME=${PHP_FPM_INSTALL_FAKETIME}
          - INSTALL_IONCUBE=${PHP_FPM_INSTALL_IONCUBE}
          - INSTALL_APCU=${PHP_FPM_INSTALL_APCU}
          - INSTALL_CACHETOOL=${PHP_FPM_INSTALL_CACHETOOL}
          - INSTALL_YAML=${PHP_FPM_INSTALL_YAML}
          - INSTALL_RDKAFKA=${PHP_FPM_INSTALL_RDKAFKA}
          - INSTALL_GETTEXT=${PHP_FPM_INSTALL_GETTEXT}
          - INSTALL_ADDITIONAL_LOCALES=${PHP_FPM_INSTALL_ADDITIONAL_LOCALES}
          - INSTALL_MYSQL_CLIENT=${PHP_FPM_INSTALL_MYSQL_CLIENT}
          - INSTALL_PING=${PHP_FPM_INSTALL_PING}
          - INSTALL_SSHPASS=${PHP_FPM_INSTALL_SSHPASS}
          - INSTALL_MAILPARSE=${PHP_FPM_INSTALL_MAILPARSE}
          - INSTALL_PCNTL=${PHP_FPM_INSTALL_PCNTL}
          - ADDITIONAL_LOCALES=${PHP_FPM_ADDITIONAL_LOCALES}
          - INSTALL_FFMPEG=${PHP_FPM_FFMPEG}
          - INSTALL_AUDIOWAVEFORM=${PHP_FPM_AUDIOWAVEFORM}
          - INSTALL_WKHTMLTOPDF=${PHP_FPM_INSTALL_WKHTMLTOPDF}
          - INSTALL_XHPROF=${PHP_FPM_INSTALL_XHPROF}
          - INSTALL_XMLRPC=${PHP_FPM_INSTALL_XMLRPC}
          - INSTALL_PHPDECIMAL=${PHP_FPM_INSTALL_PHPDECIMAL}
          - INSTALL_ZOOKEEPER=${PHP_FPM_INSTALL_ZOOKEEPER}
          - INSTALL_SSDB=${PHP_FPM_INSTALL_SSDB}
          - INSTALL_TRADER=${PHP_FPM_INSTALL_TRADER}
          - DOWNGRADE_OPENSSL_TLS_AND_SECLEVEL=${PHP_DOWNGRADE_OPENSSL_TLS_AND_SECLEVEL}
          - PUID=${PHP_FPM_PUID}
          - PGID=${PHP_FPM_PGID}
          - IMAGEMAGICK_VERSION=${PHP_FPM_IMAGEMAGICK_VERSION}
          - LOCALE=${PHP_FPM_DEFAULT_LOCALE}
          - PHP_FPM_NEW_RELIC=${PHP_FPM_NEW_RELIC}
          - PHP_FPM_NEW_RELIC_KEY=${PHP_FPM_NEW_RELIC_KEY}
          - PHP_FPM_NEW_RELIC_APP_NAME=${PHP_FPM_NEW_RELIC_APP_NAME}
          - INSTALL_DOCKER_CLIENT=${PHP_FPM_INSTALL_DOCKER_CLIENT}
          - http_proxy
          - https_proxy
          - no_proxy
      volumes:
        - ./php-fpm/php8.0.ini:/usr/local/etc/php/php.ini
        - ${APP_CODE_PATH_HOST}:${APP_CODE_PATH_CONTAINER}${APP_CODE_CONTAINER_FLAG}
        - docker-in-docker:/certs/client
      ports:
        - "9004:9004"
      expose:
        - "9000"
      extra_hosts:
        - "dockerhost:${DOCKER_HOST_IP}"
      environment:
        - PHP_IDE_CONFIG=${PHP_IDE_CONFIG}
        - DOCKER_HOST=tcp://docker-in-docker:2376
        - DOCKER_TLS_VERIFY=1
        - DOCKER_TLS_CERTDIR=/certs
        - DOCKER_CERT_PATH=/certs/client
        - FAKETIME=${PHP_FPM_FAKETIME}
      depends_on:
        - workspace
      networks:
        - backend

```
原本有含 links: 我們把它拿掉，因為build時會出錯

之後修改 NGINX Server 的 depends_on (依賴項目)

```
      depends_on:
        - php-fpm 	舊的	
        - php-fpm-8.0 	新增
```
最後再執行build
```
docker-compose up -d --build workspace nginx
```
build成功後 重啟原先的容器
```
docker-compose up -d mysql phpmyadmin
```
整個部屬好後使用nginx切換，修改./laradock/nginx/sites/ 的專案conf檔
找到 fastcgi_pass
```
初始php版本修改為
fastcgi_pass php-upstream;
變更不同版本的修改為
fastcgi_pass php-fpm-8.0:9000;	
```

修改完後要處理不同php版本的composer問題

要先進容器,要使用docker-compose 的方式進去灌不同php版本的composer
要看目前使用容器的名稱
```
 docker-compose exec php-fpm bash
 docker-compose exec php-fpm-8.0 bash 
```
進去後個別安裝composer
網址:https://getcomposer.org/download/
