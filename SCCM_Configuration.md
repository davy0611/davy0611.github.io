___
### Installing the [Windows Management Framework 5.1](https://www.microsoft.com/en-us/download/details.aspx?id=54616)

1. Check the version of PowerShell Installed
2. Download Win8.1AndW2K12R2-KB3191564-x64.msu and launch.
3. When prompted, click on the Yes button to continue
4. When prompted, click on the restart button to finalize the installation
5. Compare of the PowerShell version before and after the upgrade

![Installing_PowerShell](http://i.imgur.com/4EXKZqm.gif)

___
### Installing the Required Server Roles Using Server Manager

Overview of roles and features to be enabled/installed:

* Features
	* Background Intelligent Transfer Service (BITS)
	* .NET Framework 3.5 SP1 (or later)
	* Remote Differential Compression
	* .NET Framework 4.5.2
* Roles
	* Application Server
		* Role Services
			* .NET Framework 4.5
			* Web Server (IIS) Support
			* Windows Process Activation Service Support
				* ISAPI Extensions
				* HTTP Activation
	* Web Server
		* Role Services
			* Web Server
				* Common HTTP Features
					* Default Document
					* Directory Browsing
					* HTTP Errors
					* Static Content
					* HTTP Redirection
					* WebDAV Publishing
				* Health and Diagnostics
					* HTTP Logging
					* Logging Tools
					* Request Monitor
					* Tracing
				* Performance
					* Static Content Compresion
					* Dynamic Content Compression
				* Security
					* Request Filtering
					* Basic Authentication
					* Windows Authentication
					* Client Certificate Mapping Authority
					* Digest Authentication
					* IIS Client Certificate Mapping Authentication
					* IP and Domain Restrictions
					* URL Authorization
				* Application Development
					* .NET Extensibility 3.5
					* .NET Extensibility 4.5
					* ASP.NET 3.5
					* ASP.NET 4.5
					* ISAPI Extensions
					* ISAPI Filters
			* Management Tools
				* IIS Management Console
				* IIS 6 Management Compatibility
					* IIS 6 Metabase Compatibility
					* IIS 6 WMI Compatibility
				* IIS Management Scripts and Tools

Please be sure to launch the server manager and go through the wizard to install the server roles and features listed above. You'll want to ensure you've mounted the Server 2012 R2 ISO because you'll need to specify the directory (D:\Sources\SxS) to complete the installation of these SCCM pre-requisite server roles.

![Installing_PowerShell](http://i.imgur.com/khmXcYk.gif)

You'll need to configure Web-Dav now. I found it easier to just copy the code below and then pasting it into an administrative powershell session. Then, hit the *ENTER* key and you're all set

![Configuring WebDav](http://i.imgur.com/pKiphdW.gif)

				
```
$AppCmdDirectory = $env:SystemRoot + "\System32\inetsrv"
cd $AppCmdDirectory
.\appcmd.exe set config "Default Web Site/" /section:system.webServer/webdav/authoring /enabled:true /commit:apphost
.\appcmd.exe set config "Default Web Site/" /section:system.webServer/webdav/authoringRules /+"[Users='*',path='*',access='Read']" /commit:apphost
.\appcmd.exe set config "Default Web Site/" /section:system.webServer/webdav/authoring /properties.allowAnonymousPropfind:True /commit:apphost
.\appcmd.exe set config "Default Web Site/" /section:system.webServer/webdav/authoring /properties.allowInfinitePropfindDepth:True /commit:apphost
.\appcmd.exe set config "Default Web Site/" /section:system.webServer/webdav/authoring /properties.allowCustomProperties:false /commit:apphost
.\appcmd.exe set config "Default Web Site/" /section:system.webServer/webdav/authoring /fileSystem.allowHiddenFiles:True /commit:apphost
```

### Installing SQL Server 2014

Mount your SQL Server ISO and double-click on the Setup File. You'll want to install the Database Engine, The Reporting Services, and the Management Tools. The rest are just confirmig the defaults. One thing worth mentioning is make sure you add an administrative group when selecting your choice of authentication (i.e. Windows Authentication versus SQL authentication). Make sure you also include the machine object that corresponds to your primary site server (i.e. CM01).

![Installing SQL Server 2014](http://i.imgur.com/jBbf2Y4.gif)

___

### Configuring SQL Server 2014

Launch SQL Server 2014 Configuration Manager and the goal here is to enable IPv4 under SQL Server Network Configuration.

![Configuring SQL Server](http://i.imgur.com/1ZbA1gE.gif)

There is a known bug with the Configuration Management Console and at times it will prevent you from saving your changes. When this happens, you'll need to head on over to the registry editor and perform the changes manually; restart the SQL Server services once you've done so.

If you get an error similar to the illustration below, 

![09](http://i.imgur.com/fgwP5Qq.png)

simply follow the animation below it for your workaround

![SQL Server Management Console Bug Workaround - Large](http://i.imgur.com/npKLnIU.gif)


**OR** launch the command console in administrator mode and type the equivalent command (assuming IP4 is the one containing your production network's IP address):

```reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL12.SCCM\MSSQLServer\SuperSocketNetLib\Tcp\IP4" /v Enabled /t REG_DWORD /d 0x00000001 /f```

We also want to set a static TCP Port and avoid using dynamic ports, so it's the same management console and same locations. The goal is to clear any entries for TCP Dynamic Ports and any entries for TCP Port. Then you'll want to scroll all the way to the bottom of the bottom of the TCP/IP properties window and adding 1433 to the TCP Port entry for all addresses.

![Configuring SQL TCP Port](http://i.imgur.com/lmG1dcR.gif)

___

### Configuring SQL Server Reporting Services

If you've configured SQL Server's TCP/IP settings by:

* Enabling the IP section that contains the active IP Address used by the server (i.e. 192.168.2.101)
* Defining a Static TCP Port of 1433

Then you're ready to configure Reporting Services. Consult the animation for details.

![Configuring Reporting Services](http://i.imgur.com/a8gIV6i.gif)

