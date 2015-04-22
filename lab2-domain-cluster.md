# LAB2: Cluster Domain

Pada LAB ini kita akan membuat 4 server JBoss EAP yang akan diset sebagai sebuah **server group** atau kita sebut **cluster**, dan ke-4 server tersebut dapat dikontrol & dimonitor oleh sebuah admin console terpusat yang berada di **Domain Controllerr**.

Ke-empat server akan saling mereplikasi session data (HTTP session maupun EJB Session), replikasi message pada messaging engine dan akan dikenali oleh JBoss Web Server (dengan modul  **mod_cluster**) sebagai server yang identik. JBoss Web Server akan berlaku sebagai load balancer yang membagi trafik dari pengguna aplikasi ke semua server tersebut.

Pada LAB ini kita akan mensimulasikan ke-4 server JBoss EAP tersebut dijalankan di dua mesin terpisah. Jadi masing-masing mesin akan menjalankan 2 server instance JBoss EAP. Sedangkan JBoss Web Server dan JBoss EAP yang berfungsi sebagai Domain Controller disimulasikan berjalan di mesin lain. 
Total mesin fisik yang kita simulasikan ada 4, tetapi dalam LAB ini kita hanya akan menggunakan satu mesin PC atau laptop saja.

Setelah menyelesaikan LAB ini diharapkan anda dapat menangkap konsep kerja dan cara konfigurasi JBoss EAP sehingga dapat melakukan setup untuk jumlah server yang lebih sedikit atau lebih banyak sesuai kebutuhan dan juga bisa membuat server-group lain. Sebuah Domain Controller dapat mengelola beberapa server-group atau cluster.

Berikut gambar arsitektur dari apa yang akan kita setup:


```
                                           +-----------+
                                  ,------->| JBoss EAP | 
                                  |        | (server1) |
                                  |        |           |.......
                                  |        |           |      .
                                  +------->| JBoss EAP |      .
                                  |        | (server2) |      .    +---------------------+
                                  |        +-----------+      .    |      JBoss EAP      |
             +-----------------+  |          Mesin-A          .....| (Domain Controller) |
 Client  --->| JBoss Web Server|--+                           .    +---------------------+
(browser)    | (loadbalancer)  |  |                           .          Mesin-X
             +-----------------+  |        +-----------+      .
                  Mesin-Z         +------->| JBoss EAP |      .
                                  |        | (server3) |      .
                                  |        |           |.......
                                  |        |           |
                                  '------->| JBoss EAP |
                                           | (server4) |
                                           +-----------+
                                             Mesin-B


```
Pada Mesin-A dan Mesin-B akan dijalankan masing-masing 2 server instance JBoss EAP untuk memperlihatkan kemampuan vertical scalability. Dan semua server JBoss EAP tersebut merupakan satu kesatuan (group atau cluster) sehingga memperlihatkan kemampuan horizontal scalability.

>> Artikel ini tidak bermaksud untuk menjelaskan mengapa kita membuat arsitektur seperti itu.
>> Diharapkan anda sudah tahu konsep vertical & horizontal scalability.

Langkah-langkah besar untuk mensetup lingkungan (environment) seperti tergambar diatas yang akan kita jalankan sebagai berikut:

  1. Setup Domain Controller di Mesin-X
  2. Setup JBoss EAP di Mesin-A
  3. Setup JBoss EAP di Mesin-B
  4. Deploy aplikasi ke semua server JBoss EAP di Mesin-A dan Mesin-B lewat Domain Controller
  5. Test akses aplikasi 
  6. Setup Load Balancer/Web Server di Mesin-Z
  7. Test akses applikasi

Kebutuhan Sebelum Memulai LAB
-----------------------------

Sebelum melanjutkan untuk praktek mengikuti dokumen LAB ini, beberapa kebutuhan yang perlu disiapkan adalah sebagai berikut

- OS yang digunakanan adalah Linux, anda tidak perlu menggunakan user root
- Pada OS sudah terinstal JDK atau JRE minimal versi 1.6
- JBoss EAP versi 6.X sudah terinstal dan anda sudah tau cara menjalankan atau memberhentikan EAP server instance.



LAB: Menjalankan Domain dengan topology default
================================================

File Konfigurasi Domain
-----------------------

JBoss EAP sudah memberikan contoh file-file konfigurasi yang bisa langsung dijalankan sehingga memudahkan kita untuk melakukan setup domain. Tentu saja untuk kebutuhan nyata di lingkungan production, file konfigurasi yang ada perlu dimodifikasi sesuai kebutuhan.

File-file konfigurasi untuk setup domain ada di direktori `domain` didalam direktori dimana JBoss EAP diinstal. Isi dari direktori tersebut adalah sebagai berikut:

	```
	application-roles.properties           
	application-users.properties
	mgmt-groups.properties
	mgmt-users.properties	
	default-server-logging.properties
	domain.xml
	host-master.xml
	host-slave.xml
	host.xml
	logging.properties

	```

 - File-file dengan extension `.properties` adalah file yang menyimpan informasi credential (username, group, dan password). 
 - File `domain.xml` adalah file konfigurasi domain yang mendefiniskan module-module yang digunakan pada domain dan **mendefinisikan server-group atau cluster**
 - File `host-master.xml` adalah contoh file konfigurasi untuk sebuah mesin yang kita set sebagai Domain Controller, yaitu node yang menjadi sentral pengaturan dan tidak menjalankan server yang bekerja menerima request dari pengguna aplikasi 
 - File `host-slave.xml` adalah contoh file konfigurasi untuk sebuah mesin yang kita set sebagai application server yang akan kita deploy aplikasi dan bekerja menerima request dari pengguna aplikasi.
 - File `host.xml` adalah contoh file konfigurasi untuk sebuah mesin yang akan kita set sebagai Domain Controller tapi juga akan menjalankan application server yang akan menjalankan aplikasi.

Sebelum kita mencoba menbuat environment seperti gambar diatas, kita akan coba dulu eksplor contoh konfigurasi tersebut dengan menjalankannya __tanpa modifikasi sedikitpun__.

Kita dapat menjalankan sebuah lingkungan cluster dengan arsitektur seperti ini dengan hanya satu perintah `domain.sh` dari `<direktori_installasi_jboss_eap>/bin/`. Semua komponen yang tergambar di arsitektur dibawah ini jalan di satu mesin.


```     
                                    ,----------------------.
                                    |                      |
                                    |  +----------------+  |
                                +----->|   JBoss EAP    |  |
                                |   |  | (server-one)   |  |
+----------------------+        |   |  +----------------+  |
|      JBoss EAP       |--------+   |                      |
| (Domain Controller)  |---+    |   |  +----------------+  |
+----------------------+   |    |   |  |  JBoss EAP     |  |
                           |    +----->|  (server-two)  |  |
                           |        |  +----------------+  |
                           |        |                      |
                           |        '----------------------'
                           |            server-group:
                           |         (main-server-group)
                           |
                           |        ,----------------------.
                           |        |  +----------------+  |
                           +---------->|   JBoss EAP    |  |
                                    |  | (server-three) |  |
                                    |  +----------------+  |
                                    '----------------------'
                                           server-group:
                                       (other-server-group)
```

Sekarang ikuti langkah-langkah berikut untuk menjalankan JBoss EAP dengan arsitektur tersebut dan mengeksplor konfigurasi serta prosesnya.

1. Jalankan perintah `domain.sh`  tunggu sampai selesai proses booting dengan output terakhir seperti ini: 

    ```
    [Server:server-two] 11:55:39,476 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: JBoss EAP 6.3.0.GA (AS 7.4.0.Final-redhat-19) started in 7883ms - Started 184 of 221 services (64 services are lazy, passive or on-demand)
    ```
    
   Perintah tersebut akan membaca file konfigurasi `domain.xml` dan `host.xml` di folder `domain`
   
2. Akses admin cosole: [http://127.0.0.1:10190/](http://127.0.0.1:10190/)

3.  Lalu klik menu "Domain" > "Overview" atau akses langsung ke URL: 
    [http://127.0.0.1:10190/console/App.html#server-groups](http://127.0.0.1:10190/console/App.html#topology)

	Kita bisa lihat pada halaman tersebut beberapa server-group dan masing-masing server yang masuk dalam server-group tersebut dan juga kita bisa lihat status dari masing-masing server apakah jalan atau mati.

	> Pada EAP versi 6.3 tabel tolopology tersebut dapat diakses di menu "Runtime" lalu pilih tab TOPOLOGY

	![default server group](https://cloud.githubusercontent.com/assets/3068071/7273979/14bac12c-e923-11e4-8ee0-e5389cd0f145.png)
   
    Lihat di halaman tersebut ada 2 server-group, dengan 3 EAP servers seperti yang terlihat gambar arsitektur diatas.
   
    Kedua server di group `main-server-group` yaitu `server-one` dan `server-two` dalam keadaan hidup (jalan), karena diset otomatis jalan ketika domain dijalankan. Sedangkan `server-three` dalam keadaan mati karena memang diset untuk tidak otomatis jalan.
   
4.  Arahkan mouse pointer ke masing-masing server dalam tabel tersebut. Kita akan lihat link "Stop server" dan "Force  shutdown" untuk server yang jalan dan kita lihat link "Start server" untuk server yang dalam keadaan mati.

	Coba nyalakan `server-three` dengan mengklik link "Start server", lalu klik tombol "Confirm" pada dialog window.
   
5. Lihat file konfigurasi `domain.xml` yang ada di `<direktori_installasi_jboss_eap>/domain`, dua server-group dengan nama `main-server-group` dan `other-server-group` dapat dilihat di file tersebut dibagian paling bawah:

	```xml
	<server-groups>
	    <server-group name="main-server-group" profile="full">
	        <jvm name="default">
	            <heap size="1000m" max-size="1000m"/>
	            <permgen max-size="256m"/>
	        </jvm>
	        <socket-binding-group ref="full-sockets"/>
	    </server-group>
	    <server-group name="other-server-group" profile="full-ha">
	        <jvm name="default">
	            <heap size="1000m" max-size="1000m"/>
	            <permgen max-size="256m"/>
	        </jvm>
	        <socket-binding-group ref="full-ha-sockets"/>
	    </server-group>
	</server-groups>   
	```
	
6. Pada web admin console, klik menu "Domain" > "Server Configurations". Kita bisa lihat ada 3 server disitu yaitu `server-one`, `server-two`, dan `server-three`. 

7. Konfigurasi ke-3 server tersebut dapat dilihat di file `host.xml` yang ada di direktori `<direktori_installasi_jboss_eap>/domain`

	```
	    <servers>
	        <server name="server-one" group="main-server-group">
	            <socket-bindings port-offset="200"/>
	        </server>
	        <server name="server-two" group="main-server-group" auto-start="true">
	            <socket-bindings port-offset="350"/>
	        </server>
	        <server name="server-three" group="other-server-group" auto-start="false">
	            <socket-bindings port-offset="450"/>
	        </server>
	    </servers>
	```
	masing-masing server harus diset `port-offset`-nya karena ketiga server tersebut akan jalan di host yang sama sehingga port yang digunakan tidak boleh sama.

8. Eksplor proses Java yang dijalankan oleh JBoss EAP dengan perintah `ps ax| grep jboss`. Hasilnya anda akan lihat ada 4 proses Java seperti dibawah ini:

	```
	5770 s000  S+     0:01.31 /path/to/1.6.0.jdk/bin/java -D[Process Controller] ...
	5771 s000  S+     0:06.59 /path/to/1.6.0.jdk/bin/java -D[Host Controller] ...
	5773 s000  S+     0:13.97 /path/to/1.6.0.jdk/bin/java -D[Server:server-one] ...
	5774 s000  S+     0:13.93 /path/to/1.6.0.jdk/bin/java -D[Server:server-two] ...
	5775 s000  S+     0:13.93 /path/to/1.6.0.jdk/bin/java -D[Server:server-three] ...
    ```

    Dari list proses tersebut kita bisa melihat terdapat proses-proses Java berikut:
    
	- Process Controller: Proses ini akan dijalan pertama kali, kemudian proses ini akan menjalankan dan memastikan proses 'Host Controller' terus jalan. Jika proses Host Controller dimatikan, Proses Controller akan otomatis menghidupkannya lagi.
	- Host Controller: Proses ini adalah proses yang bertanggung jawab untuk komunikasi antara Domain Controller dengan JBoss EAP server instance yang jalan di mesin yang sama. Biasanya di setiap mesin yang akan kita setup sebagai bagian dari cluster akan memiliki satu Host Controler.
	- Server: Proses ini adalah JBoss EAP server yang akan menjalan aplikasi. 
	
9. SELESAI. Sekarang stop semua proses dengan menekan tombol CTRL+C pada terminal yang menjalankan `domain.sh`

LAB: Memulai membuat konfigurasi domain
=======================================

Sekarang mari kita mulai untuk membuat konfigurasi sesuai dengan contoh kasus yang digambarkan diawal artikel ini.

## Menyiapkan Domain Controller (MASTER) 

Kita akan mensimulasikan sebuah Domain Controller yang dijalankan di Mesin-X. Pada mesin ini tidak ada server EAP yang akan menjalankan aplikasi, hanya sebuah proses Domain Controller yang akan menjadi node untuk management dimana Management Console dijalankan. 

Langkah berikut menggunakan asumsi JBoss EAP anda diinstal di direktori `/home/jbos-eap/`

1.  Copy direktori `domain` menjadi `domain-controller` pada direktori yang sama yaitu di `/home/jboss-eap`
    ```
    cd /home/jboss-eap
    cp domain domain-controller
    ```
2.  Anda bisa hapus semua file di direktori `domain-controller/configuration/` dengan ekstensi `.xml` kecuali file `domain.xml` dan file `host-master.xml`
    
3. Jalankan Domain Controller atau master host dengan perintah berikut:
    ```
    ./bin/domain.sh -c domain.xml --host-config=host-master.xml -Djboss.domain.base.dir=domain-controller
    ```
    Ada harus berada di direktori dimana JBoss EAP diinstall, yaitu di `/home/jboss-eap`
    
    Jika anda berada di `/home/jboss-eap/bin` maka anda harus menjalankan perintah tersebut seperti ini:

    ```
    ./domain.sh -c domain.xml --host-config=host-master.xml -Djboss.domain.base.dir=../domain-controller
    ```    
    
    
    Perintah tersebut akan menjalankan Process Controller dan Host Controller tanpa menjalankan satupun Server karena berbeda dengan file konfigurasi host default yaitu `host.xml`, file host yang digunakan pada perintah tersebut yaitu `host-master.xml` tidak memiliki element `<servers>...</servers>`

4.  Buka file `host-master.xml` dengan editor atau file viewer, perhatikan disitu tidak ada elemen `<servers>...</servers>`
    Lihat juga bahwa file ini memiliki element

	```
	<domain-controller>
       <local/>
    </domain-controller>
	```
	yang artinya, ketika kita jalankan perintah diatas maka host ini akan menjadi Domain Controller.
	
	Perhatikan juga di file tersebut didefinisikan 2 server group yaitu `main-server-group` dan `other-server-group` dengan profile yang berbeda.

5.  Eksplor proses Java yang ada pada sistem dengan perintah `ps ax |grep jboss`
	Anda akan mendapatkan 2 proses berikut
	```
	...java -D[Host Controller]...
	...java -D[Process Controller]...
	```

5.  Eksplor web management console: [http://127.0.0.1:9990/](http://127.0.0.1:9990/)
    [EAP v6.4] Klik menu Domain > Overview, perhatikan tidak ada server atau server group yang terlihat di halaman tersebut.
    Lihat juga pada Domain > Server Groups, ada dua server groups di halaman tersebut seperti yang terlihat di `domain.xml`
   
    >> [EAP v6.3] dapat dilihat dimenu Domain dan Runtime > tab TOPOLOGY. 

	Port web management console tersebut didefinisikan di `host-master.xml` pada baris berikut:
	```
	 <http-interface security-realm="ManagementRealm">
        <socket interface="management" port="${jboss.management.http.port:9990}"/>
    </http-interface>
	```
    
    Nilai `${jboss.management.native.port:9999}` artinya port yang digunakan (default) adalah 9999 jika tidak dioveride dengan cara dispesifikasikan pada command line dengan opsi `-Djboss.management.native.port=XXXX` dimana XXXX adalah nilai yang akan menggantikan nilai 9999

6.  SELESAI. Kita sudah menyipkan sebuah Domain Controller pada Mesin-X.

*  Nantinya semua EAP server yang ada di Mesin-A dan Mesin-B akan mengakses (tergabung ke) Domain Controller di Mesin-X ini melalui port 9990 yaitu port yang didefinisikan sebagai __native management port__

	```
	<native-interface security-realm="ManagementRealm">
        <socket interface="management" port="${jboss.management.native.port:9999}"/>
    </native-interface>
    ```
* Setiap host yang menajalankan EAP server melakukan otentikasi ke Domain Controller saat join, oleh karena itu kita perlu membuat user untuk tiap host. Gunakan perintah `add-user.sh` untuk menambahkan user dengan realm "ManagementRealm" dan group "admin"

	1.  Untuk Mesin-A kita buat username `machinea` denagan password `Passw0rd!` dengan perintah `add-user.sh`
	    Pada akhir perintah tersebut, akan ditampilkan output seperti ini:

		```
		To represent the user add the following to the server-identities definition <secret value="UGFzc3cwcmQh" />
		```
		
		Copy secret value tersebut dan simpan karena akan kita gunakan saat mengatur (setup) Mesin-A nanti
		
	2. Untuk Mesin-B kita buat username `machineb` denagan password `Passw0rd!` dengan perintah `add-user.sh`
	   Pada akhir perintah tersebut, akan ditampilkan juga output seperti diatas, jika passwordnya sama nilai secret value akan sama. Copy secret value tersebut dan simpan karena akan kita gunakan saat mengatur (setup) Mesin-B nanti

* PENTING: Pada kondisi nyata, interface (IP address) yang di-bind untuk management haruslah menggunakan IP address yang sesungguhnya, bukan 127.0.0.1 seperti yang dispesifikasikan pada `host-master.xml` diatas. Untuk mudahnya anda bisa set agar binding dilakukan kesemua IP address yang dimiliki host yaitu dengan menggunakan IP address `0.0.0.0` seperti dibawah ini:

	```
	<interfaces>
        <interface name="management">
            <inet-address value="${jboss.bind.address.management:0.0.0.0}"/>
        </interface>
    </interfaces>
	```

## Meyiapkan JBoss EAP Server (SLAVE)

Kita akan mensimulasikan penyiapan JBoss EAP Server di 2 mesin yaitu Mesin-A dan Mesin-B yang berbeda dengan mesin dimana dijalankan Domain Controller.

Tentu di 2 mesin tersebut kita instal dahulu JBoss EAP kemudian kita ikuti langkah berikut. Pada simulasi ini kita akan menggunakan program JBoss EAP yang sama hanya saja menggunakan konfigurasi yang terpisah.

1.  Copy direktori `domain` ke direktori baru dengan nama `domain-machine-a`
 
    ```
    cd /home/jboss-eap 
    cp domain domain-machine-a
    ```
   

2.  Hapus atau ganti nama file `domain.xml` menjadi `domain.xml.backup`, agar file ini tidak dibaca saat JBoss EAP dijalankan. File tersebut tidak diperlukan pada mesin yang bertindak sebagai Slave.

3.  Edit file `host-slave.xml` yang ada di direktori `domain-machine-a`

    Tambahkan attribute `name` di element paling awal yaitu element `host` dengan value identities dari mesin, __nilai ini harus sama dengan username yang sudah ada di Domain Controller__, di langkah sebelumnya kita sudah set username `machinea` di Domain Controller, jadi element pertama menjadi seperti ini:
    
    ```
      <host name="machinea" xmlns="urn:jboss:domain:1.6">
    ``` 
Lalu set secret value untuk authentication ke Domain Controller sesuai dengan nilai yang sudah kita simpan pada langkah sebelumnya:

    ```
    ...
    <security-realm name="ManagementRealm">
      <server-identities>
         <secret value="UGFzc3cwcmQh"/>
      </server-identities> 
    ...
    ```

    Pada file ini didefinisikan alamat IP atau hostname dan port dari host-master atau Domain Controller, anda bisa lihat seperti ini. Variabel `${jboss.domain.master.address}` dapat kita set disitu atau kita spesifikasikan dengan menggunakan command line argument. Kita akan spesifikasikan pada command line argument di langkah berikutnya.
   
    ```
    <domain-controller>
       <remote host="${jboss.domain.master.address:127.0.0.1}" 
               port="${jboss.domain.master.port:9999}" 
               security-realm="ManagementRealm"/>
    </domain-controller>
    ```
    
    Karena kita akan setup 2 JBoss EAP server di Mesin-A dengan nama `server-one` dan `server-two`, maka pastikan kita memdefinisikan kedua server tersebut seperti ini:
    
    ```
    <servers>
        <server name="server-one" group="main-server-group">
            <socket-bindings port-offset="200"/>
        </server>
        <server name="server-two" group="other-server-group">
            <socket-bindings port-offset="350"/>
        </server>
    </servers>
    ```

4. Jalankan dengan perintah berikut

    ```
    ./bin/domain.sh --host-config=host-slave.xml -Djboss.domain.base.dir=domain-machine-1 -Djboss.domain.master.address=127.0.0.1
    ```
    
    Setelah dijalankan, anda akan lihat di console Mesin-X (Domain Controller) output seperti ini:
    
    ```
    [Host Controller] 00:53:56,976 INFO  [org.jboss.as.domain] (Host Controller Service Threads - 30) JBAS010918: Registered remote slave host "machine-a", JBoss EAP 6.3.0.GA (AS 7.4.0.Final-redhat-19)
    ```

	Yang menunjukan bahwa host machine-a sudah sukses bergabung (join) dengan Domain Controller
    
    >> Pada kondisi nyata dimana Domain Controler atau master-host berbeda mesin dengan mesin anggota cluster maka nilai 127.0.0.1 harus diubah dengan IP address dari master-host.

5. Lakukan langkah 1 sampai 4 sekali lagi, kali ini untuk Mesin-B dengan nama direktori `domain-machine-b` dan nama host `machineb`. Gunakan nama server yang berbeda yaitu `server-three` dan `server-four` dan juga gunakan port-offset yang berbeda misalnya 400 dan 500 karena anda akan menjalankan server tersebut di mesin laptop/PC yang sama, jadi nomor port jangan sampai bentrok.


