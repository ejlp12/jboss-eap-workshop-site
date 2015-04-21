## LAB 01 - Installasi JBoss EAP 6

1. Pastikan anda sudah memiliki (terinstal JDK atau JRE) Java 1.6 atau yang lebih tinggi
   Buka Terminal (Linux) atau Command Line (Windows) kemudian jalankan perintah
   
	```
   	java -version
   	```
   
   	Kemudian juga cek apakah variabel `JAVA_HOME` sudah diset di environment, jika belum set JAVA_HOME
   
  	```
   	export JAVA_HOME=/path/to/jdk
   	```
   	
   
2. Download [Squirrel SQL Client](http://sourceforge.net/projects/squirrel-sql/files/latest/download?source=files)

3. Login sebagai user `jboss`. Jika belum ada user ini, sebaiknya dibuat dulu dan set home directory dari user tersebut di `/home/jboss` 

	>> Sebaiknya dibuat user khusus untuk menjalankan JBoss EAP, tidak harus namanya `jboss` yang penting untuk keamanan sebaiknya yang menjalankan proses JBoss EAP bukanlah privilege user (user dengan uid 0) seperti `root`

Installasi Menggunakan GUI
==========================

1.  Jalankan perintah berikut untuk memulai instalasi menggunakan mode GUI

	```
	java -jar jboss-eap-6.4.0-installer.jar
	```

2.  Pilih bahasa yang digunakan selama instalasi, klik OK
3.  Setujui EULA, klik Next
4.  Saat diminta memasukan "installation path" ketik `/home/jboss/EAP-6.4`, klik Yes.
	Jika direktori belum ada, anda akan ditanya apakah akan dibuat direktori tersebut, klik OK.	
5.  Langkah selanjutnya akan ditampilkan paket-paket yang akan diinstal. Buka (expand) semua komponen yang ada di daftar sehingga anda bisa melihat semua komponen yang akan di instal.
	Klik Next.
6.  Saat diminta untuk membuat user, masukan `admin` sebagai username dan `Passw0rd!` sebagai password. Atau gunakan password yang lain sesuai keinginan anda
7.  Pilih Yes saat ditanya instalasi QuickStarts untuk instalasi contoh-contoh aplikasi yang bisa dideploy di JBoss EAP. Untuk environment production, anda tidak perlu instalasi QuickStarts. Klik Next 
8.  Langkah selanjutnya adalah mengatur konfigurasi Maven Repository.
    Maven diperlukan sehingga anda dapat melakukan build project yang ada di QuickStarts
9.  Langkah selanjutnya adalah menentukan port-port yang digunakan. Ada dapat memilih
	- Menggunakan default ports
	- Menentukan nilai port offset untuk semua default ports. Artinya nilai default port akan ditambahkan nilai offset.
	- Menuntukan sendiri nilai port yang akan digunakan. (Custom)
	
	Pilih opsi yang ke-3 dan pilih (checked) mengkonfigurasi port standalone dan domain. 
	
	Pada daftar port anda akan melihat pengaturan "System Property". Beberapa port yang umumnya sering kita ganti memiliki System Property yaitu nama variabel yang digunakan saat kita ingin mengganti nilai port tersebut pada saat menjalankan Jboss EAP dengan menggunakan command argument. 
	
	Misalnya kita ingin menggunakan port management-http yang berbeda, katakanlah port 9991, saat menjalankan EAP sehingga akan meng-overide setting yang ada di file konfigurasi, maka kita bisa menjalankan dengan perintah `standalone.sh -Djboss.management.http.port=9991`
	
	Klik Next beberapa kali sampai wizard port setting selesai.
		
10. Saat sudah sampai pada langkah "Server Launch", pilih  "Don't start the Server". Klik Next
11. Pada langkah "Logging Options", pilih No, untuk membiarkan pengaturan logging menggunakan konfigurasi default. Klik Next
12. Pada langkah "Configure Runtime Environment", pilih No. Klik Next dua kali sampai proses instalasi dimulai

Uninstall JBoss EAP
===================

Pada dasarnya anda dapat melakukan uninstall JBoss EAP hanya dengan menghapus direktori.

1.  Masuk ke direktori `/home/jboss/EAP-6.4/Uninstaller` dan jalankan program uninstaller

	```
	cd /home/jboss/EAP-6.4/Uninstaller
	java -jar uninstaller.jar
	```
2.  Pilih force the deletion..., klik Uninstall
3.  Klik Quit
4.  Lihat apakah direktori tempat instalasi EAP masih ada?

Installasi Menggunakan Mode Console
===================================

1.  Jalankan perintah berikut untuk memulai instalasi menggunakan mode GUI

	```
	java -jar jboss-eap-6.4.0-installer.jar -console
	```
	
2. Ikuti perintah yang ada diconsole sampai beberapa step supaya anda tau saja cara instalasi ini. Anda tidak perlu menyelesaikan cara instalasi ini, kapanpun sebelum proses instalasi selesai, cancel dengan menekan tombol CTRL+C.
   

Installasi Menggunakan ZIP File
===============================

1. Ekstrak file zip di direktori `/home/jboss/EAP-6.4`
   Sebaiknya tidak menggunakan direktori yang menggunakan special character atau memiliki white space. 
   
    ```
    unzip jboss-eap-6.4.0-installer.zip -d /home/jboss/EAP-6.4
    ```

2. Buat user administrator yang dapat melakukan manajemen terhadap JBoss EAP dengan perintah `add-user.sh`


    ```
    cd /home/jboss/EAP-6.4/bin
    ./add-user.sh
    ```
   
    Ikuti wizard-nya seperti dibawah ini:
  
	  
	```
	jboss@lab:~/EAP-6.4/bin$ ./add-user.sh

	What type of user do you wish to add?
	 a) Management User (mgmt-users.properties)
	 b) Application User (application-users.properties)
	(a):

	Enter the details of the new user to add.
	Realm (ManagementRealm) :
	Username : admin
	Password requirements are listed below. To modify these restrictions edit the add-user.properties configuration file.
	 - The password must not be one of the following restricted values {root, admin, administrator}
	 - The password must contain at least 8 characters, 1 alphabetic character(s), 1 digit(s), 1 non-alphanumeric symbol(s)
	 - The password must be different from the username
	Password :
	Re-enter Password :
	What groups do you want this user to belong to? (Please enter a comma separated list, or leave blank for none)[  ]: admin
	About to add user 'eryan' for realm 'ManagementRealm'
	Is this correct yes/no? yes
	Added user 'admin' to file '/home/jboss/EAP-6.4/standalone/configuration/mgmt-users.properties'
	Added user 'admin' to file '/home/jboss/EAP-6.4/domain/configuration/mgmt-users.properties'
	Added user 'admin' with groups admin to file '/home/jboss/EAP-6.4/standalone/configuration/mgmt-groups.properties'
	Added user 'admin' with groups admin to file '/home/jboss/EAP-6.4/domain/configuration/mgmt-groups.properties'
	Is this new user going to be used for one AS process to connect to another AS process?
	e.g. for a slave host controller connecting to the master or for a Remoting connection for server to server EJB calls.
	yes/no?
	no
	```
	
	  
3. Gunakan satu command line (non-wizard) untuk menambahkan user administrator lain
	
	```
	./add-user.sh -u admin2 -g admin -p Passw0rd! 
	```
	 
4. Lihat file `mgmt-users.properties` dan `mgmt-groups.properties` di direktori `<EAP_INSTALL_DIR>/standalone/configuration/` atau di `<EAP_INSTALL_DIR>/domain/configuration/`

5. Jalankan JBoss EAP dengan menggunakan perintah `standalone.sh` (Linux) atau `standalone.bat` (Windows) yang ada di direktori `<EAP_INSTALL_DIR>/bin/`

	```
	./standalone.sh
	```  
	
	Tunggu sampai tampil log di console seperti ini:
	
	```
	17:24:10,750 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015961: Http management interface listening on http://127.0.0.1:9990/management
17:24:10,751 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015951: Admin console listening on http://127.0.0.1:9990
17:24:10,751 INFO  [org.jboss.as] (Controller Boot Thread) JBAS015874: JBoss EAP 6.4.0.GA (AS 7.5.0.Final-redhat-21) started in 69376ms - Started 1300 of 1337 services (81 services are lazy, passive or on-demand)
	```

6. Coba akses Admin Console dari browser. Tebak sendiri URL-nya!
   Gunakan username dan password yang sudah anda buat dengan perintah `add-user.(sh|bat)`
   
   Eksplorasi menu yang ada di Admin Console.

Eksplorasi JBoss EAP
====================
	
1. Buka terminal lain, lalu lihat proses Java dari JBoss EAP dengan perintah `ps`

	```
	ps aux | grep java
	```
	
    Hasilnya adalah seperti ini:
   
	```
	jboss         1237   0.6  3.6  2625224 302740 s002  S+   Sat08AM   9:28.43 /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home/bin/java -D[Standalone] -d32 -client -verbose:gc -Xloggc:/home/jboss/EAP-6.4/standalone/log/gc.log -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=3M -XX:-TraceClassUnloading -Xms1303m -Xmx1303m -XX:MaxPermSize=256m -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headless=true -Djboss.modules.policy-permissions=true -Dorg.jboss.boot.log.file=/home/jboss/EAP-6.4/standalone/log/server.log -Dlogging.configuration=file:/home/jboss/EAP-6.4/standalone/configuration/logging.properties -jar /home/jboss/EAP-6.4/jboss-modules.jar -mp /home/jboss/EAP-6.4/modules -jaxpmodule javax.xml.jaxp-provider org.jboss.as.standalone -Djboss.home.dir=/home/jboss/EAP-6.4 -Djboss.server.base.dir=/home/jboss/EAP-6.4/standalone
	```
	
2. Lihat file log dengan menggunakan editor atau perintah `tail`. 

	```
    tail -f /home/jboss/EAP-6.4/standalone/log/server.log
	```
	
	Keluar dari perintah `tail` dengan CTRL+C
	

4. Shutdown dengan perintah `kill` atau jika menggunakan Windows anda bisa stop proses (end task) dari Window Task Manager/

	```
	kill -9 <PROCESS_ID>
	```
	
	Dimana PROCESS_ID adalah nomor proses yang ada di kolom ke-dua pada output perintah `ps`,
	pada contoh diatas PROSESS_ID adalah `1237`
	
	Jika hanya ada satu proses java, anda bisa menstop proses dengan perintah `killall java`

5. Masuk ke direktori `/home/jboss/EAP-6.4/bin` yang berisi beberapa script untuk menjalankan JBoss EAP, mengakses ke command line interface (CLI), dan juga direktori `init.d` yang menyimpan script untuk menjalankan JBoss EAP sebagai service di Linux.

	Buka dan lihat file `standalone.conf` atau di Windows file tersebut bernama `standalone.conf.bat`, lihat baris yang menunjukan setting minimum dan maximum heap size: 
	
	`JAVA_OPTS="-Xms1303m -Xmx1303m -XX:MaxPermSize=256m -Djava.net.preferIPv4Stack=true"`
	
	Kita dapat mengubah nilai Xms, Xmx dan XX:MaxPermSize sesuai kebutuhan.
	
6.  Masuk ke direktori `/home/jboss/EAP-6.4/modules`, anda akan mendapatkan banyak sekali direktori yang berisi module.
	
	Sebuah modul biasanya disimpan dalam organisasi direktori yang strukturnya mengikuti Java package, misalnya module untuk dom4j disimpan di direktori `<EAP_INSTALL_DIR>/modules/system/layers/base/org/dom4j/`. Didalam direktori terbut terdapat folder `main` dengan isi library (file JAR) dan metadata dari module tersebut yaitu file `module.xml`

	Berikut contoh isi file `module.xml`
	
	```
      <?xml version="1.0" encoding="UTF-8"?>
      <module xmlns="urn:jboss:module:1.1" name="org.dom4j">
        <properties>
          <property name="jboss.api" value="unsupported"/>
        </properties>
        <resources>
          <resource-root path="dom4j-1.6.1.redhat-6.jar"/>
              <!-- Insert resources here -->
        </resources>
        <dependencies>
            <module name="javax.api"/>
            <module name="com.sun.xml.bind"/>
            <module name="javax.xml.bind.api"/>
            <module name="org.jaxen"/>
        </dependencies>
      </module>	
	
	```
    
7.  Masuk ke direktori `welcome-content`, di direktori ini terdapat file-file default yang akan ditampilkan jika kita mengakses halaman web (default halaman web adalah http://localhost:8080)
8.  


   

   
