# LAB1

Pada LAB ini kita akan membuat 2 server JBoss EAP yang akan diset sebagai standalone HA (high available), artinya masing-masing server akan berjalan selayaknya saat kita jalankan dengan menggunakan standalone.sh/.bat atau standalone-full.sh/.bat, masing-masing server akan memiliki admin console sendiri-sendiri artinya manajemen server, deployment aplikasi dilakukan secara independent, tetapi kedua server akan saling mereplikasi session data (HTTP session maupun EJB Session) dan kedua server akan dikenali oleh JBoss Web Server (dengan modul  mod_cluster) sebagai dua server yang identik dan JBoss Web Server akan berlaku sebagai load balancer yang membagi trafik ke kedua server tersebut.

Pada LAB ini kita akan men-setup hanya dua server di mesin yang sama, tapi pada dasarnya cara yang sama dapat kita terapkan untuk jumlah server lebih dari dua dan di mesin yang berbeda.

```
                                     ,------> JBoss EAP 
                                     |        (server1)
 Client  -----> JBoss Web Server ----+
(browser)        (loadbalancer)      |
                                     '------> JBoss EAP 
                                              (server2)

```


Langkah pertama untuk membuat 2 buah server yang terkonfigurasi HA adalah menginstall JBoss EAP.  Jika kita menggunakan physical machine yang berbeda maka kita perlu menginstal JBoss EAP di masing-masing mesin tersebut. 

Instalasi JBoss EAP
-------------------

Karena kita akan menggunakan mesin yang sama, maka instalasi hanya diperlukan sekali tapi nantinya kita akan jalankan 2 server dengan konfigurasi sendiri-sendiri. Masing-masing server yang jalan di mesin yang sama akan menggunakan port yang berbeda. Lain halnya jika kita menggunakan mesin yang berbeda, kita dapat menggunakan port yang sama (sebaiknya memang kita set menggunakan port yang sama untuk kemudahaan administrasi).

1. Login sebagai `root` kemudian tambahkan user `jboss` dan set password-nya. Lalu login dengan user `jboss`
    
    ```sh
    adduser jboss
    passwd jboss
    su - jboss
    ```

2. Download installer berupa [file zip JBoss EAP versi 6.3.0](http://www.jboss.org/download-manager/file/jboss-eap-6.3.0.GA.zip)  (versi terakhir saat artikel ini dibuat)

    ```sh
    wget http://www.jboss.org/download-manager/file/jboss-eap-6.3.0.GA.zip
    ```

 3. Instal JBoss EAP di direktori `/home/jboss` dengan mengekstrak file zip

    ```sh
    unzip jboss-eap-6.3.0.GA.zip -d /home/jboss/
    ```
    


Mempersiapkan Dua Server EAP
----------------------------

Jika anda menggunakan dua mesin yang berbeda, lakukan instalasi di kedua mesin. Lalu edit file konfigurasi `standaline-ha.conf` yang ada di direktori `/home/jboss-as/jboss-eap-6.3/standalone/configuration`

1.  Buat dua folder konfigurasi untuk masing-masing server EAP.
   
    Jika anda menggunakan mesin yang sama, copy dahulu direktori `standalone/` menjadi `standalone-server1/` dan `standalone-server2/`

	```sh
	cd /home/jboss/jboss-eap-6.3/
	cp -R standalone standalone-server1
	cp -R standalone standalone-server2
	```

2.  Edit file `standalone-ha.xml` pada masing-masing direktori server1 dan server2 yaitu di `standalone-serverX/configuration/` (ganti X dengan 1 atau 2)
    
    Cari konfigurasi __modcluster__ seperti dibawah ini. Lalu ganti IP address yang tertera pada attribute __`proxy-list`__. Karena kita akan menginstal JBoss Web Server (Load balancer) pada mesin yang sama (localhost) yadi kita isi kan IP 127.0.0.1 dengan port default yaitu 6666.
    

	```xml
	<subsystem xmlns="urn:jboss:domain:modcluster:1.1">
		<mod-cluster-config advertise-socket="modcluster" sticky-session="false" connector="ajp" proxy-list="127.0.0.1:6666">
		    <dynamic-load-provider>
		        <load-metric type="busyness"/>
		    </dynamic-load-provider>
		</mod-cluster-config>
	</subsystem>
	```
	
    > Komunikasi antar server (HA) dalam cluster menggunakan protocol multicast (UDP).   
    > Jika jaringan anda tidak mendukung UDP karena diblok oleh firewall, maka anda 
    > perlu mengubah konfigurasi agar menggunakan TCP, bukan UDP. Lihat di sub bab 
    > __Cluster Network__ dibawah untuk detil mengenai konfigurasi TCP. 
	
3.  __Selesai!__ Perubahan konfigurasi JBoss EAP hanya sesederhana itu, karena file konfigurasi untuk HA (cluster) telah tersedia pada default instalasi. Kita bisa menggunakan `standalone-ha.xml` atau `standalone-full-ha.xml` untuk tujuan LAB ini.

    Sekarang kita jalankan kedua server tersebut. 
    
    Jalankan server pertama
    
    ```sh
    ./bin/standalone.sh -b 0.0.0.0 -u 230.0.0.4 -c standalone-ha.xml -Djboss.server.base.dir=standalone-server1 -Djboss.node.name=server1 -Djboss.socket.binding.port-offset=100
    ```
    
    Sekarang jalankan server kedua di console/terminal yang lain:
    
    ```sh
    ./bin/standalone.sh -b 0.0.0.0 -u 230.0.0.4 -c standalone-ha.xml -Djboss.server.base.dir=standalone-server2 -Djboss.node.name=server2 -Djboss.socket.binding.port-offset=200
    ```
    
    Karena kita menggunakan mesin yang sama, agar port dari kedua server tersebut tidak bentrok kita perlu menggunakan port-offset yang berbeda.

    Berikut penjelasan mengenai opsi yang digunakan pada perintah diatas
 

	* `-b` : Binding address. 0.0.0.0 artinya port akan di-binding kesemua network interface/IP address yang dimiliki mesin tersebut.
	* `-u` : Multicast address. Multicast address digunakan oleh mod_cluster dari JBoss EAP untuk memberikan informasi kepada load balancer.
	* `-Djboss.node.name` : Nama node. Nama ini dapat juga diset pada file `standalone.xml ` misalnya `<server name="server1" xmlns="urn:jboss:domain:1.5">`
	* `-Djboss.socket.binding.port-offset` : Default port yang akan digunakan nilainya akan ditambahkan dengan nilai offset ini. Misalnya default port 8080 akan menjadi
	  8180 (8080+100) jika nilai offset adalah 100
	  
4.  Untuk memudahkan menjalankan kedua server tersebut, kita buat script berikut:
 
    Buat file startserver1.sh di direktori instalasi `/home/jboss/jboss-eap-6.3/`

	```
	!/bin/bash
	./bin/standalone.sh \
	-b 0.0.0.0 \
	-u 230.0.0.4 \
	-c standalone-ha.xml \
	-Djboss.server.base.dir=standalone-server1 \
	-Djboss.node.name=server1 \
	-Djboss.socket.binding.port-offset=100
	```

    Set file tersebut menjadi executable

    ```sh
    chmod 755 startserver1.sh

    ```
    
    Untuk server ke-2, buat file startserver2.sh di direktori instalasi jboss-eap, lalu     
    ganti server1 menjadi server2 dan ganti nilai port-offset

	```
	cp startserver1.sh startserver2.sh
	```
	
	Isi dari file `startserver2.sh` adalah seperti berikut:

	```	
	./bin/standalone.sh \
	-b 0.0.0.0 \
	-u 230.0.0.4 \
	-c standalone-ha.xml \
	-Djboss.server.base.dir=standalone-server2 \
	-Djboss.node.name=server2 \
	-Djboss.socket.binding.port-offset=200
	```


### Melihat Konfigurasi HA dari server EAP

Sebelum kita lanjut dengan instalasi JBoss Web Server (Apache HTTP Server) dan module mod_clusternya. Saya akan bahas terlebih dahulu mengenai konfigurasi HA dari server EAP.

Kemudahan konfigurasi HA pada LAB ini karena sudah tersedianya file konfigurasi HA bawaan dari EAP yaitu file `standalone-ha.xml` atau `standalone-full-ha.xml`. Lalu apa bedanya file tersebut dengan file `standalone.xml` yang tanpa HA? Kita akan bandingkan sekarang. Silakan buka file `standalone-ha.xml` lagi

#### Extension Module

Untuk konfifurasi HA, dibutuhkan extensions tambahan berikut ini:

	```
    <extensions>
        <extension module="org.jboss.as.clustering.infinispan"/>
        <extension module="org.jboss.as.clustering.jgroups"/>
        ...
    </extensions>
    ```
    
[Infinispan](http://infinispan.org/) adalah software untuk in-memory data grid yang digunakan JBoss EAP untuk menyimpan data HTTP/EJB session, cache

[JGroups](http://www.jgroups.org/) adalah software yang digunakan untuk komunikasi antar server/node dalam satu cluster

#### Replikasi Session

Replikasi session pada konfigurasi HA secara default sudah terkonfigurasi seperti dibawah ini. Replikasi session dilakukan dengan mode REPL artinya session akan direplikasi ke semua server dalam cluster. 

Keterangan mengenai detail konfigurasi bisa dilihat di [dokumentasi](https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.1/html/Development_Guide/chap-Clustering_in_Web_Applications.html#Enable_Session_Replication_in_Your_Application)

```xml
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
                <transport lock-timeout="60000"/>
                <replicated-cache name="repl" mode="ASYNC" batching="true">
                    <eviction strategy="LRU" max-entries="10000"/>
                    <file-store/>
                </replicated-cache>
                <replicated-cache name="remote-connector-client-mappings" mode="SYNC" batching="true"/>
                <distributed-cache name="dist" l1-lifespan="0" mode="ASYNC" batching="true">
                    <eviction strategy="LRU" max-entries="10000"/>
                    <file-store/>
                </distributed-cache>
            </cache-container>
            ...
        </subsystem>
```
> CATATAN: Agar web session direplikasi, aplikasi web yang dideploy perlu ditambahkan tag **`<distributable/>`** di file `web.xml` seperti ini
>
    
    <web-app  xmlns="http://java.sun.com/xml/ns/j2ee"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
                              http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd" 
          version="2.4">
        <distributable/>
    </web-app>
    



#### Cluster Network

Konfigurasi JGroups untuk clustering dapat dilihat dibawah ini. Dengan mekanisme UDP ini, setting load balancer menjadi otomatis. Bisa dibilang JBoss Web Server dengan mod_cluster tidak perlu dikonfigurasi manual untuk mengetahui masing-masing IP address.

Bisa kita lihat konfigurasi JGroups dibawah ini. Konfigurasi ini sudah sedikir dimodifikasi, versi aslinya anda tidak akan menemukan blok element `<stack name="tcp">`. Blok elemen ini digunakan jika komunikasi antar server dan dari server ke load balancer tidak bisa menggunakan multicast (UDP). Untuk menggunakan protokol TCP pada komunikasi cluster kita perlu mengubah nilai dari attribute `default-stack` pada element `subsystem` dibawah ini menjadi __`tcp`__

```xml
        <subsystem xmlns="urn:jboss:domain:jgroups:1.1" default-stack="udp">
            <stack name="udp">
                <transport type="UDP" socket-binding="jgroups-udp"/>
                <protocol type="PING"/>
                <protocol type="MERGE3"/>
                <protocol type="FD_SOCK" socket-binding="jgroups-udp-fd"/>
                <protocol type="FD"/>
                <protocol type="VERIFY_SUSPECT"/>
                <protocol type="pbcast.NAKACK"/>
                <protocol type="UNICAST2"/>
                <protocol type="pbcast.STABLE"/>
                <protocol type="pbcast.GMS"/>
                <protocol type="UFC"/>
                <protocol type="MFC"/>
                <protocol type="FRAG2"/>
                <protocol type="RSVP"/>
            </stack>
            <stack name="tcp">
                <transport type="TCP" socket-binding="jgroups-tcp"/>
                <protocol type="MPING" socket-binding="jgroups-mping"/>
                <protocol type="TCPPING">
                    <property name="initial_hosts">0.0.0.0[7600],0.0.0.0[7600]</property>
                    <property name="num_initial_members">2</property>
                    <property name="port_range"> 0</property>
                    <property name="timeout">
                        2000
                    </property>
                </protocol>
                <protocol type="MERGE2"/>
                <protocol type="FD_SOCK" socket-binding="jgroups-tcp-fd"/>
                <protocol type="FD"/>
                <protocol type="VERIFY_SUSPECT"/>
                <protocol type="pbcast.NAKACK"/>
                <protocol type="UNICAST2"/>
                <protocol type="pbcast.STABLE"/>
                <protocol type="pbcast.GMS"/>
                <protocol type="UFC"/>
                <protocol type="MFC"/>
                <protocol type="FRAG2"/>
                <protocol type="RSVP"/>
            </stack>
        </subsystem>
```

Jika anda menggunakan protocol TCP dan menggunakan mesin yang berbeda, ganti nilai element `property` dengan name `initial_host` sesuai IP address dari masing-masing mesin, misalnya `192.168.0.12[7600],192.168.0.13[7600]`.

Nilai 7600 adalah port dari __jgroups-tcp__ yang dispesifikasikan pada element `socket-binding-group`

#### Socket Binding

Konfigurasi yang berbeda pada mode standalone HA (cluster) yang lain adalah pada blok socket binding. Pada konfigurasi ini terdapat konfigurasi untuk port yang digunakan untuk mekanisme clustering.

Jika kita ingin penggunaan TCP port, bukan multicast (UDP), maka pastikan pada file konfigurasi anda memiliki konfigurasi binding dengan nama `jgourps-tcp` dan `jgroups-tcp-fd` seperti dibawah ini:

```xml
    <socket-binding-group name="standard-sockets" default-interface="public" port-offset="${jboss.socket.binding.port-offset:0}">
        ....
        <socket-binding name="jgroups-mping" port="0" multicast-address="${jboss.default.multicast.address:230.0.0.4}" multicast-port="45700"/>
        <socket-binding name="jgroups-tcp" port="7600"/>
        <socket-binding name="jgroups-tcp-fd" port="57600"/>
        <socket-binding name="jgroups-udp" port="55200" multicast-address="${jboss.default.multicast.address:230.0.0.4}" multicast-port="45688"/>
        <socket-binding name="jgroups-udp-fd" port="54200"/>
        <socket-binding name="modcluster" port="0" multicast-address="224.0.1.105" multicast-port="23364"/>
        ....
    </socket-binding-group>
```

Instalasi dan Konfigurasi JBoss Web Server dan mod_cluster
----------------------------------------------------------

Untuk platform Windows lihat di sub-bab setelah ini.

>> PERHATIAN: Default port dari HTTP server mod_cluster yang didownload adalah 6666. Jika anda menggunakan browser Google Chrome, anda akan mendapatkan error __ERR_UNSAFE_PORT__ karena port tersebut dianggap tidak aman. Agar port tersebut tidak diblok maka anda dapat menambahkan opsi misalnya `–explicitly-allowed-ports=6666,6667` pada shortcut dari aplikasi Chrome.

Saat penulisan artikel ini, versi terakhir JBoss Web Server adalah versi 2.1.0 (mod_cluster 1.3.1) dan kita akan gunakan versi tersebut dalam LAB ini.

Download Apache HTTP Server dari JBoss Web Server versi Enterprise dari website [Red Hat](https://access.redhat.com/jbossnetwork/restricted/listSoftware.html?downloadType=distributions&product=webserver&productChanged=yes) sesuai dengan platform yang and a pakai. Untuk LAB ini silakan download file dari link __Red Hat JBoss Web Server 2.1.0 Apache HTTP Server for RHEL 7 x86_64__

Jika tidal memiliki Red Hat support subscription, anda bisa juga download versi community-nya di link berikut:

http://mod-cluster.jboss.org/downloads/1-3-1-Final-bin

Download file yang sudah termasuk Apache HTTP Server (httpd) dan sesuai platform yang dan gunakan misalnya [`mod_cluster-1.3.1.Final-linux2-x64-ssl.tar.gz`](http://downloads.jboss.org/mod_cluster//1.3.1.Final/linux-x86_64/mod_cluster-1.3.1.Final-linux2-x64-ssl.tar.gz) untuk "mod_cluster native budles with httpd and openssl"

1.  Login sebagai root lalu ekstrak file installer di direktori `/`
	Program Apache httpd akan diinstall di `/opt/jboss/httpd` lalu jalankan dengan perintah `apachectl start` 

    ```
    tar zxvf mod_cluster-1.3.1.Final-macosx-x64.tar.gz -C /
    cd /opt/jboss/httpd/bin/
    ./apachectl start
	```

2.  Modul mod_cluster secara default sudah terinstal dan terkonfigurasi sebagai modul Apache HTTP server. 
    Kita sekarang bisa cek JBoss EAP cluster yang sudah terdeteksi oleh  Apache HTTP server dengan mengakses 
    ke URL berikut:
    
    [http://localhost:6666/mod_cluster_manager](http://localhost:6666/mod_cluster_manager)

### Troubleshooting mod_cluster

1. Cek konfigurasi mod_cluster di file `/opt/jboss/httpd/httpd/conf/httpd.conf`

   ```
   # MOD_CLUSTER_ADDS
   # Adjust to you hostname and subnet.
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

       <Location /mod_cluster_manager>
        SetHandler mod_cluster-manager
         Require ip 127.0.0
       </Location>

     </VirtualHost>
  </IfModule>
    ```

   * Pastikan IP address dan port di baris `AdvertiseGroup 224.0.1.105:23364` sesuai dengan IP dan port mod_cluster yang diset di JBoss EAP:

     ```
     <socket-binding name="modcluster" port="0" multicast-address="224.0.1.105" multicast-port="23364"/>
     ```

   * Tambahkan `AllowDisplay On` agar bisa kita cek apakah modul yang perlu digunakan untuk cluster sudah berhasil di-load.
     Anda dapat lihat di [http://localhost:6666/mod_cluster_manager](http://localhost:6666/mod_cluster_manager) seperti ini:

     ![image](https://cloud.githubusercontent.com/assets/3068071/11315832/a00e2368-902b-11e5-811e-7a438e8d526f.png)

   * Pastikan ada baris `EnableMCPMReceive`

2. Cek Advertise protocol yang menggunakan UDP apakah sudah jalan dengan perintah `tcpdump -n host 224.0.1.105`, seperti berikut.

   Kita akan lihat paket UDP dikirim ke IP multicast 224.0.1.105 port 23364 (default port)
   

   ```
   $ sudo tcpdump -n host 224.0.1.105
   tcpdump: data link type PKTAP
   tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
   listening on pktap, link-type PKTAP (Packet Tap), capture size 65535 bytes
   07:03:16.564750 IP 192.168.1.4.23364 > 224.0.1.105.23364: UDP, length 302
   07:03:20.512593 IP 192.168.1.4 > 224.0.1.105: igmp v2 report 224.0.1.105
   07:03:21.708928 IP 192.168.1.4.23364 > 224.0.1.105.23364: UDP, length 302
   07:03:26.850105 IP 192.168.1.4.23364 > 224.0.1.105.23364: UDP, length 302
   07:03:31.997529 IP 192.168.1.4.23364 > 224.0.1.105.23364: UDP, length 302
   ^C
   5 packets captured
   45 packets received by filter
   0 packets dropped by kernel
   ```
   

Mod_cluster di Windows
----------------------

Download mod_cluster untuk windows x86

[mod_cluster-1.3.1.Final-windows-x86.zip](http://downloads.jboss.org/mod_cluster//1.3.1.Final/windows/mod_cluster-1.3.1.Final-windows-x86.zip)

1. Ekstrak file `mod_cluster-1.3.1.Final-windows-x86.zip` di direktori --misalnya-- `D:\httpd-2.2`
2. Buka command prompt dengan user Administrator (run as Administrator)
3. Masuk ke direktori tersebut dan jalankan `installconf.bat`

	```
	D:\
	cd httpd-2.2\bin
  installconf.bat
	```
4. Jalankan Web Server dengan perintah `httpd.exe`
5. Modul mod_cluster secara default sudah terinstal dan terkonfigurasi sebagai modul Apache HTTP server. 
  Kita sekarang bisa cek JBoss EAP cluster yang sudah terdeteksi oleh  Apache HTTP server dengan mengakses ke URL berikut:
    
    [http://localhost:6666/mod_cluster_manager](http://localhost:6666/mod_cluster_manager)

    ![Tampilan mod_cluster_manager](https://cloud.githubusercontent.com/assets/3068071/7254245/ff17c642-e86b-11e4-8206-2be8ddcdb2c8.png)

Test HA cluster
---------------

Kita akan test HA Cluster dengan beberapa scenario:

  - Load balancing HTTP request. Kita akan gunakan browser untuk mengakses suatu aplikasi ke IP & port dari HTTP server (load balancer). Kita harapkan load balancer akan mengarahkan request ke masing-masing server JBoss EAP secara seimbang (load balance)
  - Failed over HTTP request. Kita akan matikan salah satu server JBoss EAP lalu akses aplikasi IP & port dari HTTP server (load balancer). Kita harapkan load balancer dapat selalu mengarahkan request ke server yang hidup, sehingga kita tidak pernah mendapatkan error karena tidak dapat terkoneksi ke server yang dimatikan. 
  - Kita juga akan lihat bahwa session antar server JBoss EAP akan saling terreplikasi.



1. Untuk keperluan test, kita perlu mendeploy suatu aplikasi di masing-masing server JBoss EAP. Download aplikasi WAR dari URL berikut:
   
   ```
   cd /home/jboss-as/
   wget https://github.com/ejlp12/jboss-eap-workshop-site/raw/master/resources/cluster-test.war
   ```
   
2. Deploy file `cluster-test.war` ke masing-masing server JBoss EAP dengan meng-copy file ke direktory `deployment`

  ```
  cp cluster-test.war jboss-eap-6.3/standalone-server1/deployment/
  cp cluster-test.war jboss-eap-6.3/standalone-server2/deployment/
  ```

3. Test masing-masing aplikasi dengan cara mengakses langsung URL dari JBoss EAP (bukan URL load-balancer)
   Buka browser dan akses kedua URL berikut:
   
   - [http://localhost:8180/cluster-test](http://localhost:8180/cluster-test)
   - [http://localhost:8280/cluster-test](http://localhost:8280/cluster-test)
   
   Lihat tampilan yang muncul di browser, nama dari "nodeId" tiap server yang diakses berbeda yaitu "server1" dan "server2"
   Refresh masing-masing halaman tersebut, dan perhatikan jumlah session ("# of requests placed on session").
   
4. Test aplikasi dengan cara mengakses URL load-balancer yaitu 
   
   [http://localhost/cluster-test](http://localhost/cluster-test)
   
   Perhatikan lagi "nodeId" dan jumlah session.
   
5. Stop salah satu server, misalnya server1 dengan cara berikut:
   
   ```sh
   cd /home/jboss-as/jboss-eap-6.3/bin
   ./jboss-cli.sh
   ```
   
   Setelah masuk ke mode command line inteface (CLI) dengan mendapatkan prompt `[disconnected /]`
   Jalankan perintah berikut
   
   ```
   connect localhost:10199
   shutdown
   ```
   
   > Port default untuk management adalah 9999, port management pada server1 adalah 10199 (9999 + 100)
   > karena offset port server1 adalah 100
   
   Akses halaman [http://localhost:6666/cluster-test](http://localhost:6666/cluster-test) secara berkali-kali dan 
   perhatikan nama dari "nodeId" untuk melihat efek dari matinya server1.
   





