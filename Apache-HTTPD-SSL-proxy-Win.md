#Setup Apache HTTP Server as Proxy for SAP Mobile Documents


##Setup for Windows
Windows doesn't come pre-installed with Apache2 wih SSL support, :(

Open your browser and go to [link](http://httpd.apache.org/download.cgi#apache24)

Download & Install the package for Windows - **Win32 Binary including OpenSSL 0.9.8y (MSI Installer)**

##Post Installation
Generally, Apache2 is installed in the below location

	C:\Program Files(x86)\Apache Software Foundation\Apache2.x


Open your text editor in **Administartor** mode

Open the file C:\Program Files(x86)\Apache Software Foundation\Apache2.x\conf\httpd.conf


######Edit *httpd.conf* as below


Step 1 : Uncomment the below line to include the SSL binary

	LoadModule ssl_module libexec/apache2/mod_ssl.so

Step 2 : Change the Listen port to 9006 or any other port (optional)

Step 3 : Change the server name to localhost:9006

Step 4 : Uncomment the below line to include the configuration file specific to HTTPS


	Include C:\Program Files(x86)\Apache Software Foundation\Apache2.x\conf\extra\httpd-ssl.conf


Now, Open the file C:\Program Files(x86)\Apache Software Foundation\Apache2.x\conf\extra\httpd-ssl.conf
######Edit *httpd-ssl.conf* file as below


Step 1 : Change the Listen port to 9005 or any other port

Step 2 : Edit the VirtualHost host and port
	
	<VirtualHost *:9005>

Step 3 : Enable SSLEngine and SSLProxyEngine

	SSLEngine on
	SSLProxyEngine on

Step 4 : Add the location of your SSLCertificateFile ans SSLCertificateKeyFile

Let us add it to the root folder of apache2

	SSLCertificateFile "C:\Program Files(x86)\Apache Software Foundation\Apache2.x\conf\server.crt"
	
	SSLCertificateKeyFile "C:\Program Files(x86)\Apache Software Foundation\Apache2.x\conf\server.key"

Step 5 : If you want to route any calls outside the corporate firewall, add them to the ProxyRemoteMatch with the proxy location
	
Route all **/mcm/** calls through the given proxy server
	
	ProxyRemoteMatch /mcm/ http://proxy.wdf.sap.corp:8080

Step 6 : Enable ProxyVia, Disable ProxyRequests

	ProxyVia On
	ProxyRequests Off
	
Step 7 : Now add the locations to be proxied

	
	ProxyPass /mcm/ https://smd-box.neo.ondemand.com/mcm/
	ProxyPassReverse /mcm/ https://smd-box.neo.ondemand.com/mcm/
	
	ProxyPass /ca.pca.manager/ http://localhost:8080/ca.pca.manager/
	ProxyPassReverse /ca.pca.manager/ http://localhost:8080/ca.pca.manager/

######Time to set up the server keys

Step 1 : Open command prompt, navigate to Desktop
	
	cd ~/Desktop

Step 2 : Generate a Private Key

	openssl genrsa -des3 -out server.key 1024
	
Step 3 : Generate a CSR (Certificate Signing Request)
	
	openssl req -new -key server.key -out server.csr
	
	Country Name: IN
	State: KA
	Locality Name: BLR
	Organisation Name: SAP
	Organisation Unit Name: AI
	Common Name: localhost
	Email Address: <sap email address>
	
	Challenge password - Leave it empty
	Optional Company name - Leave it empty
	
Step 4 : Remove Passphrase from Key

	cp server.key server.key.org
	openssl rsa -in server.key.org -out server.key
	
Step 5 : Generating a Self-Signed Certificate
	
	openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
	
	
Step 6 : Installing the Private Key and Certificate

	cp server.crt C:\Program Files(x86)\Apache Software Foundation\Apache2.x\server.crt
	cp server.key C:\Program Files(x86)\Apache Software Foundation\Apache2.x\server.key
	
###### Note : If command prompt throws error with openssl not found / bad command, navigate to
C:\Program Files(x86)\Apache Software Foundation\Apache2.x\bin\

You should be able to find the openssl.exe in this folder.
You can generate the certificates here & copy them to 
	
	C:\Program Files(x86)\Apache Software Foundation\Apache2.x\conf
	
Step 7 : Restart Apache HTTP Server
	
	sudo apachectl stop
	sudo apachectl start

###Test if the proxy works
Open web browser and navigate to https://localhost:9005/ca.pca.manager/

(If your tomcat is running, you will be able to see the Manager Person Application)

