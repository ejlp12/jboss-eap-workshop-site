Untuk menyiapkan JBoss EAP server agar dapat diakses dengan SSL/HTTPS, ada dua cara implementasi, yaitu:

* Menggunakan implementasi Java 
* Menggunakan implementasi native berbasis OpenSSL
In both cases you have to configure keys and (self-signed) certificates for your web server. This guide will briefly explain how to accomplish that for both options.

Pure Java SSL-Setup using keytool
We will generate a secret key/certificate and store it in a file called a "key store". The certificate is valid for 30 years = 10950 days. The password use for encryption is "secret".

One important issue is the common name (CN) of the certificate. For some reason this is referred to as "first and last name". It should however match the name of the web server, or some browsers like IE will claim the certificate to be invalid although you may have accepted it already.

Step 1: Generate key
 $ keytool -genkey -alias foo -keyalg RSA -keystore foo.keystore -validity 10950
Enter keystore password: secret
Re-enter new password: secret
What is your first and last name?
  [Unknown]:  foo.acme.com
What is the name of your organizational unit?
  [Unknown]:  Foo
What is the name of your organization?
  [Unknown]:  acme corp
What is the name of your City or Locality?
  [Unknown]:  Duckburg
What is the name of your State or Province?
  [Unknown]:  Duckburg
What is the two-letter country code for this unit?
  [Unknown]:  WD
Is CN=foo.acme.com, OU=Foo, O=acme corp, L=Duckburg, ST=Duckburg, C=WD correct?
  [no]:  yes

Enter key password for <deva> secret
    (RETURN if same as keystore password):  
Re-enter new password: secret
Step 2: Configure JBoss
<subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="false">
  <connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"  redirect-port="443" />
 
  <connector name="https" scheme="https" protocol="HTTP/1.1" socket-binding="https" enable-lookups="false" secure="true">
    <ssl name="foo-ssl" password="secret" protocol="TLSv1" key-alias="foo" certificate-key-file="../standalone/configuration/foo.keystore" />
  </connector>
  ...
</subsystem>
Native SSL-Setup using OpenSSL
Again we will generate a private key and a self-signed certificate. Additionally, you can also export the certificate to a pkcs12 format file.
You can import it into the Windows certificate storage if you have problems with the Internet Explorer.

Step 1: Generate key
$ openssl genrsa -des3 -out foo.pem 1024
Generating RSA private key, 1024 bit long modulus
............++++++
...................++++++
e is 65537 (0x10001)
Enter pass phrase for foo.pem: secret
Verifying - Enter pass phrase for foo.pem: secret
Step 2: Generate certificate
$ openssl req -new -x509 -key foo.pem -out foo-cert.pem -days 10950
Enter pass phrase for foo.pem: secret
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:WD
State or Province Name (full name) [Some-State]:Duckburg
Locality Name (eg, city) []:Duckburg
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Acme Corp
Organizational Unit Name (eg, section) []:Foo
Common Name (eg, YOUR name) []:foo.acme.com
Email Address []:
Step 3: (Optional) Generate PKCS12 file
$ openssl pkcs12 -export -in foo-cert.pem -inkey foo.pem  -out foo.p12
Enter pass phrase for foo.pem: secret
Enter Export Password: secret
Verifying - Enter Export Password: secret
Step 4: Configure JBoss
<subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="true">
  <connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http" redirect-port="443" />
 
  <connector name="https" scheme="https" protocol="HTTP/1.1" socket-binding="https" enable-lookups="false" secure="true">
    <ssl name="foo-ssl" password="secret" certificate-key-file="../standalone/configuration/foo.pem" certificate-file="../standalone/configuration/foo-cert.pem"/>
  </connector>
  ...
Port configuration
The above example assumes that you have configured JBoss to use the standard ports 80 (HTTP) and 443 (HTTPS). Accesses to the HTTP port will be redirected HTTPS.

<socket-binding-group name="standard-sockets" default-interface="public" ...>
<socket-binding name="http" port="80" />
<socket-binding name="https" port="443" />
...
</socket-binding-group>
Labels:
