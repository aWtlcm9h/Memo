# Dark web上のサイト構築備忘録  

### 目次
- [目的](#目的)  
- [環境](#環境)  
- [Webサイトの構築](#Webサイトの構築)  
- [Hidden Serviceの設定](#Hidden Serviceの設定)  
- [自己署名証明書](#自己署名証明書)  


#### 目的  
- Dark Web上にPHPの動作するサイトを構築  
- 自己署名証明書でhttpsに対応(暗号化によって出口ノードでの監視を潜り抜ける，証明書発行時のホスト名登録によるホスト名流出を防ぐ)  

#### 環境  
Raspberry Pi3 ModelB  

#### Webサイトの構築  
アップデート  

```$ sudo apt-get update```  
```$ sudo apt-get upgrade -y```  

tor，nginx，phpのインストール  

```$ sudo apt-get install tor```  
```$ sudo apt-get install nginx```  
```$ sudo apt-get install php7.3-fpm```  

nginxの起動確認  

```$ sudo /etc/init.d/nginx start```  

nginxの設定ファイルのバックアップ&編集  

```$ sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.bak```  
```$ sudo vi /etc/nginx/sites-available/default```  

44行目を編集  
編集前
> index index.html index.htm index.nginx-debian.html;  

編集後
> index index.html index.htm index.nginx-debian.html index.php;

56-61,63行目を編集  
> location ~ \.php$ {  
>    include snippets/fastcgi-php.conf;
>  
>    \# With php-fpm (or other unix sockets):  
>    fastcgi_pass unix:/var/run/php/php7.3-fpm.sock;  
>    \# With php-cgi (or other tcp sockets):  
> \#       fastcgi_pass 127.0.0.1:9000;  
> }  

php.iniの編集  

```$ sudo vi /etc/php/7.3/fpm/php.ini```  

793行目(cgi.fixpathinfo)を編集　

編集前
> ;cgi.fixpathinfo=1  

編集後
> cgi.fixpathinfo=0  

php&nginxの動作確認(下記実行後，localhost/test.phpにブラウザからアクセス)  

```$ sudo /etc/init.d/php7.3-fpm restart```  
```$ sudo /etc/init.d/nginx restart```  
```$ echo -n '<?php phpinfo(); ?>' > /var/www/html/test.php```  

#### Hidden Serviceの設定  

torrcファイルの71,72行目を編集("### This section is just for location-hidden services ###"のある行らへん)  
```$ sudo vi /etc/tor/torrc```  

編集前
> \#HiddenServiceDir /var/lib/tor/hidden_service/  
> \#HiddenServicePort 80 127.0.0.1:80  

編集後
> HiddenServiceDir /var/lib/tor/hidden_service/  
> HiddenServiceVersion 3
> HiddenServicePort 80 127.0.0.1:80  

torを再起動  
```$ sudo service tor restart```  

生成されたホスト名の確認  
```$ sudo cat /var/lib/tor/hidden_service/hostname```  


#### 自己署名証明書  

作成(localhostに当たる部分は上で生成したhostnameの中身を指定)  
```openssl req -x509 -sha256 -nodes -days 3650 -newkey rsa:2048 -subj /CN=localhost -keyout server.key -out server.crt```  

上で作成した証明書を移動(sslディレクトリが無かったら作成)  

```$ sudo mv ./server.key /etc/nginx/ssl/```  
```$ sudo mv ./server.crt /etc/nginx/ssl/```  
```$ sudo chmod 600 /etc/nginx/ssl/*```  

nginxの設定ファイルをhttps用に編集する  
```$ sudo vi /etc/nginx/sites-available/default```  

22,23行目  
変更前  
> listen 80 default_server;  
> listen [::]:80 default_server;  

変更後  
> \# listen 80 default_server;  
> \# listen [::]:80 default_server;  

25行目(# SSL configuration)の下あたりに追記  
> listen 443 ssl default_server;  
> listen [::]:443 ssl default_server;  

40行目らへんに追記  
> ssl_certificate_key /etc/nginx/ssl/server.key;  
> ssl_certificate /etc/nginx/ssl/server.crt;  
> ssl_protocols TLSv1.2;  
> ssl_prefer_server_ciphers on;  

ここまで完了せてhttpsで接続を試みるとhttpsで接続出来るはず....  
※自己署名証明書なので端末に証明書を入れてもブラウザによっては”危険を承知で続行？”みたいなのは出る  
