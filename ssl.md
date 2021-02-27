# 如何在bitnami 下申請wordpress網站 數字證書（数字凭证）
    

    
bitnami安裝數字證書說明文檔：https://docs.bitnami.com/google/how-to/generate-install-lets-encrypt-ssl/#alternative-approach

*本視頻是NGINX 服務器*https://youtu.be/m1zeN29hyhM

### 如果是 NGINX 服務器
#### 第一步：
Install The Lego Client
```
cd /tmp
curl -Ls https://api.github.com/repos/xenolf/lego/releases/latest | grep browser_download_url | grep linux_amd64 | cut -d '"' -f 4 | wget -i -
tar xf lego_vX.Y.Z_linux_amd64.tar.gz
sudo mkdir -p /opt/bitnami/letsencrypt
sudo mv lego /opt/bitnami/letsencrypt/lego
```


#### 第二步：
關閉所有服務
```
sudo /opt/bitnami/ctlscript.sh stop
```

申請數字證書
```
sudo /opt/bitnami/letsencrypt/lego --tls --email="lirangfeng000@gmail.com" --domains="lirangguo.xyz" --domains="www.lirangguo.xyz" --path="/opt/bitnami/letsencrypt" run
```
*注意：替換 EMAIL-ADDRESS 成您自己的郵箱  替換 DOMAIN 成您的域名*


#### 第三步：
配置服務器使用數字證書

```
sudo mv /opt/bitnami/nginx/conf/server.crt /opt/bitnami/nginx/conf/server.crt.old
sudo mv /opt/bitnami/nginx/conf/server.key /opt/bitnami/nginx/conf/server.key.old
sudo mv /opt/bitnami/nginx/conf/server.csr /opt/bitnami/nginx/conf/server.csr.old
sudo ln -sf /opt/bitnami/letsencrypt/certificates/lirangguo.xyz.key /opt/bitnami/nginx/conf/server.key
sudo ln -sf /opt/bitnami/letsencrypt/certificates/lirangguo.xyz.crt /opt/bitnami/nginx/conf/server.crt
sudo chown root:root /opt/bitnami/nginx/conf/server*
sudo chmod 600 /opt/bitnami/nginx/conf/server*
```
*替換 DOMAIN 成您的域名 有兩行要替換喔  例如： DOMAIN.key 替換成 您的域名.key* 千萬要改喔！



打開所有服務
```
sudo /opt/bitnami/ctlscript.sh start
```
#### 第四步：

打開https://你的域名  看看是否有一個小鎖🔒



## 如何强制使用SSL（如何自动跳转到https）
###安装好数字证书，一开始chrom 提示是一个小锁🔒，但是点到其他页面又提示不安全，虽然是https://ilileyou.gq 但是还是有不安全的提醒。是怎么回事呐？
我们要到 SSH界面下（黑色的界面）

### 第一句 编辑worpress 配置文件
```
sudo vi /opt/bitnami/apps/wordpress/htdocs/wp-config.php
```
键盘输入 i 可编辑状态
键盘上下左右 找到
```
define('WP_SITEURL', 'http://' . $_SERVER['HTTP_HOST'] . '/');
define('WP_HOME', 'http://' . $_SERVER['HTTP_HOST'] . '/');
```
替换成
```
define('WP_SITEURL', 'https://' . $_SERVER['HTTP_HOST'] . '/');
define('WP_HOME', 'https://' . $_SERVER['HTTP_HOST'] . '/');
```
#### 保存wp-config.php
 键盘输入
##### 按 ESC 键

#####  :wq      回车

## 自動更新數字證書
創建自動執行腳本
```
sudo vi /opt/bitnami/letsencrypt/scripts/renew-certificate.sh
```
複製內容
```
#!/bin/bash

sudo /opt/bitnami/ctlscript.sh stop nginx

sudo /opt/bitnami/letsencrypt/lego --tls --email="lirangfeng000@gmail.com" --domains="lirangguo.xyz" --domains="www.lirangguo.xyz" --path="/opt/bitnami/letsencrypt" renew --days 90
sudo /opt/bitnami/ctlscript.sh start nginx
```


使得腳本可以執行

```
sudo chmod +x /opt/bitnami/letsencrypt/scripts/renew-certificate.sh
```

执行以下命令以打开crontab编辑器
```
sudo crontab -e
```
将以下行添加到crontab文件并保存：
```
0 0 1 * * /opt/bitnami/letsencrypt/scripts/renew-certificate.sh 2> /dev/null
```

### 第二句重启服务：

```
sudo /opt/bitnami/ctlscript.sh restart
```
