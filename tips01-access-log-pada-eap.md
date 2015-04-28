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
                   <directory path="${jboss.server.name}-access.log" relative-to="jboss.server.log.dir" />
                </access-log>
            </virtual-server>
        </subsystem>
```
Secara default access log akan disimpan di `${jboss.server.log.dir}/default-host` dimana `default-host` adalah nama yang 
dispesifikasikan di elemen `virtual-server` jika kita tidak menspesifikasikan letak log file dengan elemen `<directory path="">`.

Pada contoh diatas file akan disimpan dengan mana `abc-access.log`
