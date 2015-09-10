Replikasi web session dan EJB session (statefull session bean/sfsb) sudah diset pada mode domain dan juga jika kita menggunakan mode ha (standalone-ha.xml atau standalone-full-ha.xml)

Object session secara default disimpan di memory cache menggunakan module [Infinispan](http://infinispan.org/)

Kita bisa lihat di file konfigurasi setting replikasi session seperti ini:
```
<subsystem xmlns="urn:jboss:domain:infinispan:1.4">
             ...
            <cache-container name="web" aliases="standard-session-cache" default-cache="repl" module="org.jboss.as.clustering.web.infinispan">
                <transport lock-timeout="60000"/>
                <replicated-cache name="repl" mode="ASYNC" batching="true">
                    <file-store/>
                </replicated-cache>
                <replicated-cache name="sso" mode="SYNC" batching="true"/>
                <distributed-cache name="dist" l1-lifespan="0" mode="ASYNC" batching="true">
                    <file-store/>
                </distributed-cache>
            </cache-container>
            <cache-container name="ejb" aliases="sfsb sfsb-cache" default-cache="repl" module="org.jboss.as.clustering.ejb3.infinispan">
            ...
            ...
```

Infinispan sebagai module atau system yang menyimpan session, menggunakan [JGroups](http://www.jgroups.org/) library untuk komunikasi yang handal (reliable messaging) antara JBoss EAP server nodes. JGroup mendukung dua jenis protocol network yaitu:

* UDP (with IP multicast)
* TCP

Untuk server nodes yang berada di satu segment network (subnet) anda bisa menggunakan UDP, sedangkan jika segment yang berbeda maka gunakan TCP. Konfigurasi UDP atau TCP bisa dilihat konfigurasi pada dibagian berikut:

```xml
<cache-container name="web" default-cache="repl"> 
  <transport stack="tcp"/> 
  . . . .
</cache-container>
```

sedangkan konfigurasi detail penggunaan protocol bisa dilihat dibagian:

```xml
<subsystem xmlns="urn:jboss:domain:jgroups:1.1" default-stack="udp">
  <stack name="udp">
    <transport type="UDP" socket-binding="jgroups-udp" diagnostics-socket-binding="jgroups-diagnostics"/>
    <protocol type="PING"/>
    <protocol type="MERGE2"/>
    . . . . .
  </stack>
  <stack name="tcp">
    <transport type="TCP" socket-binding="jgroups-tcp" diagnostics-socket-binding="jgroups-diagnostics"/>
    <protocol type="MPING" socket-binding="jgroups-mping"/>
    <protocol type="TCPPING">
        <property name="initial_hosts">192.168.1.100[7600],192.168.2.200[7600]</property>
    . . . . .
  </stack>
</subsystem>
```

## Konfigurasi pada aplikasi

Selain itu dari sisi aplikasi web yang akan dideploy perlu ditambahkan tag <distributable/> di file web.xml seperti ini agar  web session direplikasi:

```xml
<web-app  xmlns="http://java.sun.com/xml/ns/j2ee"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
      xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
                          http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd" 
      version="2.4">
    <distributable/>
</web-app>
```
