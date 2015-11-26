Untuk menyiapkan JBoss EAP server agar dapat diakses dengan SSL/HTTPS, ada dua cara implementasi, yaitu:

* Menggunakan implementasi Java 
* Menggunakan implementasi native berbasis OpenSSL

Pada dasarnya keduanya sama saja hanya saja format penyimpanan certificate yang berbeda. Untuk kedua opsi tersebut anda harus membuat (self-signed) certificate.


## Menyiapkan SSL certificate versi Java mengunakan keytool

Anda perlu men-generate sebuah  secret key/certificate untuk kemudian akan disimpan di sebuah file yang disebut "key store". 

> One important issue is the common name (CN) of the certificate. For some reason this is referred to as "first and last name". It should however match the name of the web server, or some browsers like IE will claim the certificate to be invalid although you may have accepted it already.

1. Generate key
   
   Anggap tempat instalasi JBoss EAP di direktori `/Servers/EAP-6.4`, kita akan membuat keystore file dengan nama `ejlp12.keystore`
   ```
 $ cd /Servers/EAP-6.4
 $ keytool -genkey -alias ejlp12-alias -keyalg RSA -keystore ejlp12.keystore -validity 10950
Enter keystore password: secret
Re-enter new password: secret
What is your first and last name?
  [Unknown]:  host1.ejlp12.com
What is the name of your organizational unit?
  [Unknown]:  JBoss
What is the name of your organization?
  [Unknown]:  EJLP12 corp
What is the name of your City or Locality?
  [Unknown]:  Jakarta
What is the name of your State or Province?
  [Unknown]:  Jakarta
What is the two-letter country code for this unit?
  [Unknown]:  ID
Is CN=host1.ejlp12.com, OU=JBoss, O=EJLP12 corp, L=Jakarta, ST=Jakarta, C=ID correct?
  [no]:  yes

Enter key password for <deva> P@ssw0rd
    (RETURN if same as keystore password):  
Re-enter new password: P@ssw0rd
```

2. Configure JBoss

   Edit dile `standalone.xml` atau `domain.xml` jika EAP akan dijalankan dalam mode domain (edit `profile` yang anda inginkan)

   ```
    <subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="false">
      <connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"  redirect-port="443" />
      ...
      <connector name="https" scheme="https" protocol="HTTP/1.1" socket-binding="https" enable-lookups="false" secure="true">
         <ssl name="foo-ssl" password="P@ssw0rd" protocol="TLSv1" key-alias="ejlp12-alias" certificate-key-file="ejlp12.keystore" />
      </connector>
   ...
   </subsystem>
   ```
   
   Pastikan juga terdapat `socket-binding` yang definisikan port `https`
   ```
      <socket-binding-group name="..." default-interface="public">
          ...
          <socket-binding name="http" port="8080"/>
          <socket-binding name="https" port="8443"/>
   ```
3. Selesai, restart JBoss EAP dan test 

   Mode standalone: 
   ```
   ./bin/jboss-cli.sh -c --command=reload
   ```
    
  Mode domain, di host domain controller:
  ```
  ./bin/jboss-cli.sh -c --controller=localhost:9990 --command=reload
  ``` 
    
  Test dengan akses ke https://hostname:8443/ 

## Native SSL-Setup menggunakan OpenSSL

1.  Generate key
   ```
   $ cd /Server/EAP-6.4
   $ openssl genrsa -des3 -out foo.pem 1024
   Generating RSA private key, 1024 bit long modulus
   ............++++++
   ...................++++++
   e is 65537 (0x10001)
   Enter pass phrase for foo.pem: secret
   Verifying - Enter pass phrase for foo.pem: secret
   ```
2. Generate certificate

   ```
   $ openssl req -new -x509 -key ejlp12.pem -out ejlp12-cert.pem -days 10950
   Enter pass phrase for foo.pem: P@ssw0rd
   You are about to be asked to enter information that will be incorporated
   into your certificate request.
   What you are about to enter is what is called a Distinguished Name or a DN.
   There are quite a few fields but you can leave some blank
   For some fields there will be a default value,
   If you enter '.', the field will be left blank.
   -----
   Country Name (2 letter code) [AU]:ID
   State or Province Name (full name) [Some-State]:Jakarta
   Locality Name (eg, city) []:Jakarta
   Organization Name (eg, company) [Internet Widgits Pty Ltd]: EJLP12 Corp
   Organizational Unit Name (eg, section) []: JBoss
   Common Name (eg, YOUR name) []: host1.ejlp12.com
   Email Address []:
   ```

3. (Optional) Generate PKCS12 file
   ```
   $ openssl pkcs12 -export -in ejlp12-cert.pem -inkey ejlp12.pem  -out ejlp12.p12
   Enter pass phrase for host1.ejlp12.com: P@ssw0rd
   Enter Export Password: P@ssw0rd
   Verifying - Enter Export Password: P@ssw0rd
   ```

4. Configure JBoss 

   ```
   <subsystem xmlns="urn:jboss:domain:web:1.1" default-virtual-server="default-host" native="true">
     <connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http" redirect-port="443" />
    
     <connector name="https" scheme="https" protocol="HTTP/1.1" socket-binding="https" enable-lookups="false" secure="true">
       <ssl name="foo-ssl" password="P@ssw0rd" certificate-key-file="ejlp12.pem" certificate-file="ejlp12-cert.pem"/>
     </connector>
     ...
   ```
   
Selanjutnya lakukan langkah yang sama seperti anda men-setup key-store versi Java diatas.
