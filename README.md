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
找個資料夾git clone Laradock  
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
將DATA_PATH_HOST=~/.laradock/data修改為DATA_PATH_HOST=./data  
(沒調整的話 mysql or mariadb 的 container 可能會無法啟動)


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



