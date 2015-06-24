Secara default JBoss EAP akan menambahkan informasi versi JSP engine di HTTP header dalam variabel `X-Powered-By` jika client melakukan request ke URL sebuah halaman JSP. Berikut contoh response dari JBoss EAP.

```
HTTP/1.1 200 OK
X-Powered-By: JSP/2.2
Set-Cookie: JSESSIONID=mGuR3DVr6LyhEPZtyNFDvKHD; Path=/SimpleJspServletDB
Content-Type: text/html;charset=EUC-KR
Content-Length: 321
Date: Wed, 24 Jun 2015 21:33:46 GMT
Server: ejlp-server
```

Untuk menghilangkan informasi tersebut dapat dilakukan penambahan `jsp-configuration` dengan atribute `x-powered-by="false"` seperti berikut

```xml
<subsystem xmlns="urn:jboss:domain:web:1.5" default-virtual-server="default-host" native="false">
  <connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"/>
  <virtual-server name="default-host" enable-welcome-root="false">
    <alias name="localhost"/>
  </virtual-server>
  <configuration>
    <jsp-configuration x-powered-by="false"/>
  </configuration>
</subsystem>
```

perubahan diatas juga dapat dilakukan dengan CLI

```
/subsystem=web/configuration=jsp-configuration/:write-attribute(name=x-powered-by,value=false)
```

Penambahan header `X-Powered-By` dilakukan pada saat JSP di-compile, jadi jika JSP sudah di-compile sebelum perubahan konfigurasi diatas kita akan masih melihat adanya header tersebut. Karena itu sebaiknya kita menghapus cache sehingga semua file JSP akan di-compile ulang. Caranya dengan menghapus semua folder di direktori `standalone/tmp/` atau `domain/servers/<nama_server>/tmp/`

```
cd <JBOSS_HOME>/standalone/tmp
rm -rf *
```

## X-Powered-By dari konten JSF

Jika konten yang diakses adalah JSF, bisa jadi header `X-Powered-By` akan tetap muncul, untuk menghilangkannya dapat ditambahkan konfigurasi berikut pada file `WEB-INF/web.xml` yang ada di dalam file aplikasi

```xml
<context-param>
    <param-name>com.sun.faces.sendPoweredByHeader</param-name>
    <param-value>false</param-value>
</context-param>
```
