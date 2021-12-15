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
 docker-compose up -d nginx mariadb workspace   
 docker-compose up -d apache phpmyadmin         
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
@SET /P host=Enter your project name:

@echo its:%host%

@cd C:\Windows\System32\drivers\etc

@echo 127.0.0.1       %host%.test  >>  hosts

@cd C:\Users\rock\Desktop\dockertest\laradock\apache2\sites

@echo ^<VirtualHost *:80^>                              >>  default.apache.conf   
@echo   ServerName %host%.test                          >>  default.apache.conf 
@echo   DocumentRoot /var/www/%host%                    >>  default.apache.conf 
@echo   Options Indexes FollowSymLinks                  >>  default.apache.conf 
@echo    ^<Directory "/var/www/%host%"^>                >>  default.apache.conf 
@echo       AllowOverride All                           >>  default.apache.conf 
@echo      ^<IfVersion ^< 2.4 ^>                         >>  default.apache.conf 
@echo           Allow from all                          >>  default.apache.conf 
@echo      ^</IfVersion^>                               >>  default.apache.conf 
@echo      ^<IfVersion ^>^= 2.4^>                        >>  default.apache.conf 
@echo           Require all granted                     >>  default.apache.conf 
@echo      ^</IfVersion^>                               >>  default.apache.conf 
@echo    ^</Directory^>                                 >>  default.apache.conf 
@echo   ErrorLog /var/log/apache2/error.log             >>  default.apache.conf 
@echo   CustomLog /var/log/apache2/access.log combined  >>  default.apache.conf 
@echo  ^</VirtualHost^>                                 >>  default.apache.conf 
docker ps -a
docker restart laradock_apache2_1
```
bat檔寫好後
右鍵以系統管理員身分執行
輸入你測試網站的名稱


