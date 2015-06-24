Jika ada error pada saat menjalankan suatu class atau halaman JSP biasanya JBoss EAP akan menampilkan error stacktrace pada response yang dapat dilihat di browser. Hal ini dapat memberikan informasi yang mungkin memudahkan peretas untuk melakukan kejahatan pada server kita. 

Untuk men-disable stacktrace pada response dari JBoss EAP dapat ditambahkan konfigurasi berikut:

```xml
<configuration>
  <jsp-configuration display-source-fragment="false"/>
</configuration>
```

atau dengan CLI

```
cd <JBOSS_HOME>/bin/jboss-cli.sh

[disconnected /] connect localhost
[localhost /]/subsystem=web/configuration=jsp-configuration:write-attribute(name=display-source-fragment, value=false)
```
