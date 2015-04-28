Biasanya kita mendapatkan access log yang menunjukan HTTP request/response di Web/Proxy Server. Tapi seringkali kita menjalankan 
JBoss EAP server untuk langsung diakses end-users, jadi tidak ada Web Server atau Proxy Server yang menjadi jembatan sebelum 
request diterima EAP. EAP secara default tidak mengeluarkan access log, tapi kita bisa membuat agar EAP dapat mengeluarkan access
log yang serupa dengan Apache HTTPD server. Caranya adalah sebagai berikut:

Tambahkan konfigurasi ini di `standalone.xml`

```
<server name="abc" xmlns="urn:jboss:domain:1.3">
...
    <subsystem xmlns="urn:jboss:domain:web:1.2" default-virtual-server="default-host" native="false">
            <connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"/>
            <virtual-server name="default-host" enable-welcome-root="true">
                <alias name="localhost"/>
                <alias name="example.com"/>
               <access-log pattern='%h %l %u %t %r %s %b %{Referer}i %{User-Agent}i %S %T'>
                   <directory path="${jboss.server.name}-accessLog" relative-to="jboss.server.log.dir" />
                </access-log>
            </virtual-server>
        </subsystem>
```
Secara default access log akan disimpan di `${jboss.server.log.dir}/default-host` dimana `default-host` adalah nama yang 
dispesifikasikan di elemen `virtual-server` jika kita tidak menspesifikasikan letak log file dengan elemen `<directory path="">`.

Pada contoh diatas letak dan mana file-nya akan seperti ini `<EAP_INSTALL_DIR>/standalone/log/abc-accessLog/access_log.2015-04-27` 
dengan contoh format lognya adalah seperti ini:

```
127.0.0.1 - - [27/Apr/2015:15:08:09 +0700] GET /queryrunner/theme/style.css HTTP/1.1 404 1116 http://localhost:8080/queryrunner/index.jsp Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36 f5P0X2cu4+zQaUM2i+AXSMRw 0.095
127.0.0.1 - - [27/Apr/2015:15:15:01 +0700] GET /queryrunner/index.jsp HTTP/1.1 200 586 - Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36 vfab6BqFD4608V+vzSL80Fmw 0.066
```

Apa yang akan di-log pada file tersebut dapat kita spesifikasikan pada attribute `pattern`. Informasi lebih detail untuk
membuat pattern tersebut bisa dibaca di [http://docs.jboss.org/jbossweb/7.0.x/config/valve.html](http://docs.jboss.org/jbossweb/7.0.x/config/valve.html)
