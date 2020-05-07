# MockRestService

A simple solution to mock network traffic as HTTP POST/GET request. It is also an example of utilizing simple RestService library. https://github.com/stanislawbartkowski/RestService

Can be easily extended to more HTTP methods.<br><br>
The solution is provided as two Intellij Idea projects: the server and the client. The server is a standalone Java application, the only prerequisite is RestService library mentioned above.It does not require any web server container like Tomcat or Jetty.

The client is Python 3 script.
# Server
## Installation
Download and install RestService to the local Maven repository.<br>
https://github.com/stanislawbartkowski/RestService

> git clone git clone https://github.com/stanislawbartkowski/MockRestService.git<br>
> mvn clean package<br>
> ls target
```
MockRestService-1.0-SNAPSHOT.jar
MockRestService-1.0-SNAPSHOT-jar-with-dependencies.jar
```
## Run HTTP non-secure MockRest server
> cp template/runserver.sh .<br>
```
PORT=9800
java -cp target/MockRestService-1.0-SNAPSHOT-jar-with-dependencies.jar com.org.mockrestservice.MockRestService $POR
```
The only configuration is to specify the port number (here 9800).
> ./runserver.sh
```
Feb 29, 2020 7:42:20 PM com.rest.restservice.RestLogger info
INFO: Start HTTP Server, listening on port 9800
Feb 29, 2020 7:42:20 PM com.rest.restservice.RestLogger info
INFO: Register service: resetcounter
Feb 29, 2020 7:42:20 PM com.rest.restservice.RestLogger info
INFO: Register service: counter
Feb 29, 2020 7:42:20 PM com.rest.restservice.RestLogger info
INFO: Register service: rest
Feb 29, 2020 7:42:20 PM com.rest.restservice.RestLogger info
INFO: Register service: upload

```
The server is ready.

## Test

* POST request
> curl -X POST  http://localhost:9800/rest?content=Hello

* Upload
> curl -X POST -d Hello  http://localhost:9800/upload

There is an internal counter scoring the number of POST "rest" requests received
* Reset counter
> curl -X GET  http://localhost:9800/resetcounter
* The current value of the counter
> curl -X GET  http://localhost:9800/counter

## Run HTTPS secure MockRest server

### Configuration
Copy and customize *template/secure.properties* file containing data for server certificates.

| Variable | Description | Example |
| -------- | -------- | ----------|
| store.key.filename | Pathname of the server key file |  cert/mykey.keystore
| key.store.password | Password protecting key file | secret
| alias | Alias of the server certificate in the key file | alias

In the *runserver.sh* script file uncomment the setting of *SECURITY* variable. The variable should point to *secure.properties* file.<br> 
### Self-signed certificate
Example command to prepare self-signed certificate. Pay attention to proper key bit size.<br>
> keytool -genkey -alias alias -keypass secret -keystore mykey.keystore -storepass secret -keyalg rsa -keysize 2048

Test, -k switches off the certificate validation<br>
> curl -X POST https://localhost:9800/rest?content=Hello -k

### Server certificate requested by CSR file
Generate self-signed certiticate<br>
> keytool -genkey -alias alias -keypass secret -keystore mykey.keystore -storepass secret -keyalg rsa -keysize 2048<br>

Generate CSR file<br>
> keytool -certreq -keyalg RSA -alias alias -file certreq.csr -keystore mykey.keystore -keyalg rsa -keysize 2048 <br>

Send CSR file to CA center for signing. <br>
Important: In case of authority certificate chain, it is necessary to import all certificates in the chain separately one after one. Otherwise, while importing the server certificate it will fail with "Failed to establish chain from reply."<br>
<br>
Import root certificate<br>
> keytool -import -alias root -keystore mykey.keystore -file root.cert.pem<br>

Import intermediate certificate<br>
> keytool -import -alias intermediate -keystore mykey.keystore -file intermediate.cert.pem<br>

Import server certificate<br>
> keytool -import -alias alias -keystore mykey.keystore -file root.cert.pem<br>

Test with certificate validation. The hostname (thinkde.sb.com) shoud match the CN of the server certificate.

> curl -X POST https://thinkde.sb.com:9800/rest?content=Hello --cacert ca-chain.cert.pem<br>

## Kerberos
MockRestService can be also protected by Kerberos power. In order to setup Kerberos authentication, both client and server need access to KDC/AD service.

### Prepare service principal and keytab
Assume the MockRestService is installed on *thinkde.sb.com* host. Kerberos is FQN aware, it is important that hostname resolves to FQN. The service principal name should follow the pattern *HTTP/\<FQN host name\>@\<realm name\>". Example here: *HTTP/thinkde.sb.com@CENTOS.COM.REALM*
  
> kadmin <br>

> addprinc -randkey HTTP/thinkde.sb.com@CENTOS.COM.REALM<br>
> ktadd -k service.keytab HTTP/thinkde.sb.com@CENTOS.COM.REALM
```
Entry for principal HTTP/thinkde.sb.com@CENTOS.COM.REALM with kvno 3, encryption type aes256-cts-hmac-sha1-96 added to keytab WRFILE:service.keytab.
Entry for principal HTTP/thinkde.sb.com@CENTOS.COM.REALM with kvno 3, encryption type aes128-cts-hmac-sha1-96 added to keytab WRFILE:service.keytab.
kadmin:
```
### Prepare JAAS file for MockRestService
JAAS file should contain *server* entry allowing passwordless service authentication.






# Client
## Prerequisites
* Python 3 (tested with Python 3.6 level)<br>
> yum install python36<br>

* *requests* package
> yum install python36-pip<br>
> python36 -m pip install requests<br>

## Run unit test

Modify the hostname and the application name in Tomcat server.

> cd MockRest/CallRest/test/rest<br>
> vi Test1.py<br>
```
SERVERHOST="localhost:8080"
APPNAME="RestMockServer"
```
Run the test.<br>

> cd MockRest/CallRest/test/rest<br>
> PYTHONPATH=../..   python36 -m unittest Test1.py 
```
/home/user/rest/MockRest/CallRest/test/rest/Test1.py:18: ResourceWarning: unclosed file <_io.TextIOWrapper name='/home/user/rest/MockRest/CallRest/com/rest/../../resource/f.txt' mode='r' encoding='UTF-8'>
  res = R.uploadFile("f.txt")
<html><head></head><body>File f.txt uploaded successfully.<br><a href="UploadDownloadFileServlet?fileName=f.txt">Download f.txt</a></body></html>
./home/user/rest/MockRest/CallRest/test/rest/Test1.py:22: ResourceWarning: unclosed file <_io.TextIOWrapper name='/home/user/rest/MockRest/CallRest/com/rest/../../resource/upgrade-error.txt' mode='r' encoding='UTF-8'>
  res = R.uploadFile("upgrade-error.txt")
<html><head></head><body>File upgrade-error.txt uploaded successfully.<br><a href="UploadDownloadFileServlet?fileName=upgrade-error.txt">Download upgrade-error.txt</a></body></html>
.received
.
----------------------------------------------------------------------
Ran 3 tests in 0.021s

OK

```
## Run Rest/HTTP client
> cd MockRest/CallRest<br>
> PYTHONPATH=. python36 com/MainRun.py<br>

* run.sh Single run
* runattack.sh Launch a seriers of  *run.sh* in parallel
