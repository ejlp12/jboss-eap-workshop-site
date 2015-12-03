# Konfigurasi HTTPS/SSL pada JBoss EAP

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

## Native SSL-Setup menggunakan OpenSSL (Alternatif)

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


## Konfigurasi JBoss EWS HTTP Server + mod_cluster menggunakan HTTPS/SSL **DRAFT**

> Catatan dibawah ini belum sepnuhnya jalan! MASIH PERLU DIREVISI.

Buat certificate files di direktori `/Servers/EAP-6.4/`

```
$ cd /Servers/EAP-6.4/

$ openssl rsa -passin pass:x -in server.pass.key -out server.key
writing RSA key

$ rm server.pass.key

$ openssl req -new -key server.key -out server.csr
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
Organization Name (eg, company) [Internet Widgits Pty Ltd]:EJLP12 corp
Organizational Unit Name (eg, section) []:JBoss
Common Name (e.g. server FQDN or YOUR name) []:host1.ejlp12.com
Email Address []:ejlp12@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:P@ssw0rd
An optional company name []:EJLP12

$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
Signature ok
subject=/C=ID/ST=Jakarta/L=Jakarta/O=EJLP12 corp/OU=JBoss/CN=host1.ejlp12.com/emailAddress=ejlp12@gmail.com
Getting Private key

$  ls -la
total 3
-rw-r--r--   1 ejlp12  wheel   1.3K Nov 27 05:02 server.crt
-rw-r--r--   1 ejlp12  wheel   1.1K Nov 27 05:02 server.csr
-rw-r--r--   1 ejlp12  wheel   1.6K Nov 27 05:00 server.key
```

Uncomment beberapa baris berikut di `httpd.conf`

```
LoadModule ssl_module /opt/jboss/httpd/lib/httpd/modules/mod_ssl.so
LoadModule socache_shmcb_module /opt/jboss/httpd/lib/httpd/modules/mod_socache_shmcb.so
ServerName host1.ejlp12.com:80
```

Buka komentar (uncomment) baris berikut sehingga HTTP Server akan dapat diases melalui https (port 433):
```
Include conf/extra/httpd-ssl.conf
```

Ubah juga bagian konfigurasi mod_cluster di `httpd.conf` agar komunikasi port 6666 menggunakan SSL:

```
<IfModule manager_module>
  Listen 127.0.0.1:6666
  ManagerBalancerName mycluster
  <VirtualHost 127.0.0.1:6666>
    <Location />
     Require ip 127.0.0
    </Location>

    KeepAliveTimeout 300
    MaxKeepAliveRequests 0
    #ServerAdvertise on http://@IP@:6666
    AdvertiseFrequency 5
    #AdvertiseSecurityKey secret
    #AdvertiseGroup @ADVIP@:23364
    AdvertiseGroup 224.0.1.105:23364
    EnableMCPMReceive
    AllowDisplay On

   SSLEngine on
   SSLCipherSuite AES128-SHA:ALL:!ADH:!LOW:!MD5:!SSLV2:!NULL
   SSLVerifyDepth 10
   SSLProxyEngine On
   SSLCertificateKeyFile /Servers/EAP-6.4/server.key
   SSLCertificateFile /Servers/EAP-6.4/server.crt
   SSLCACertificateFile /Servers/EAP-6.4/server.csr
   LogLevel debug

    <Location /mod_cluster_manager>
       SetHandler mod_cluster-manager
       Require ip 127.0.0
    </Location>

  </VirtualHost>
</IfModule>
```

File `conf/extra/httpd-ssl.conf` akan membutuhkan certificate yang berada di direktori `/opt/jboss/httpd/httpd/conf/`, karena kita simpan file certificate-nya di `/Servers/EAP-6.4/` maka saya buat symlink berikut:

```
ln -s /Servers/EAP-6.4/server.crt /opt/jboss/httpd/httpd/conf/server.crt
ln -s /Servers/EAP-6.4/server.key /opt/jboss/httpd/httpd/conf/server.key
```

Restart Apache HTTP Server
```
sudo /Servers/MOD_CLUSTER/jboss/httpd/sbin/apachectl restart
```

Test koneksi https dengan mengakses menggunakan browser ke https://HTTP_SERVER_HOST

Check error log:
```
$ tail -f /Servers/MOD_CLUSTER/jboss/httpd/httpd/logs/error_log
```


### Konfigurasi mod_cluster pada Domain Mode menggunakan HTTPS/SSL

Ubah  `domain.xml` dengan menambahkan konfigurasi ssl seperti berikut:

```
           <subsystem xmlns="urn:jboss:domain:modcluster:1.2">
               <!-- // Tutup bagian ini, ganti konektor dari ajp ke https
                <mod-cluster-config advertise-socket="modcluster" connector="ajp">
                -->
                <mod-cluster-config advertise-socket="modcluster" connector="https">
                    <dynamic-load-provider>
                        <load-metric type="busyness"/>
                    </dynamic-load-provider>
                    <ssl password="password" protocol="TLSv1" key-alias="foo" certificate-key-file="foo.keystore" />
                </mod-cluster-config>
            </subsystem>
```

> NOTE: Apa harus ditambahkan `proxy-list`??? menjadi seperti ini:
>
> `<mod-cluster-config advertise-socket="modcluster" connector="https" proxy-list="IP_PROXY_HTTPS_SERVER1:443,IP_PROXY_HTTPS_SERVER2:443">`

Tambahakan `proxy-name="IP_PROXY_HTTPS_SERVER" proxy-port="443"`

```
   <subsystem xmlns="urn:jboss:domain:web:2.2" ...>
      ...
      <connector name="https" scheme="https" protocol="HTTP/1.1" socket-binding="https" enable-lookups="false" secure="true" proxy-name="IP_PROXY_HTTPS_SERVER" proxy-port="443">
```

Restart semua servers

Referensi: [https://access.redhat.com/solutions/199493](https://access.redhat.com/solutions/199493)


### Check mod_cluster

Check [https://localhost:6666/mod_cluster_manager](https://localhost:6666/mod_cluster_manager), gunakan **https** BUKAN http.
