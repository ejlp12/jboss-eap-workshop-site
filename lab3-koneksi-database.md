# Mengkonfigurasi Koneksi JBoss EAP ke ProsgreSQL Database

>> Tutorial ini dibuat dengan menggunakan JBoss EAP versi 6.3.0, cara yang sama untuk koneksi ke database lainnya misalnya
MySQL, Oracle Database, MariaDB akan sama saja. Yang membedakan adalah library JDBC, connection URL, driver class, dan 
datasource class.

Sebelum melakukan konfigurasi JBoss EAP agar terkoneksi ke database, pertama tentu perlu dipastikan bahwa database sudah
terinstall dan dapat diakses dari server EAP. 

Untuk dapat terkoneksi ke database, biasanya aplikasi menggunakan __datasource__ yang diakses menggunakan __JNDI__.
Datasource tersebut memberikan koneksi database kepada aplikasi. Datasource menggunakan __connection-pool__ yang mengelola
koneksi ke database agar lebih efisien. Tiap objek koneksi yang ada di connection-pool menggunakan __JDBC driver__ untuk 
membuatkoneksi ke database. Setiap database memiliki JDBC driver-nya masing-masing dan datasource perlu diset agar objek 
koneksi bisa dibuat saat dibutuhkan.

Konfigurasi datasource bisa dilihat dan dibuat lewat Web Management Console di menu Configuration > Connector - Datasources

<img src="https://cloud.githubusercontent.com/assets/3068071/7330740/a4b9ab42-eb1e-11e4-9b93-91aadcef55f8.png" height="500" width="500" >


Datasource membutuhkan JDBC driver, suatu JAR file yang bisa dikenali oleh EAP dengan dua cara yaitu:

 1. Men-deploy JDBC JAR file sebagai aplikasi. Cara ini lebih direkomendasikan karena lebih mudah dan dapat dengan mudah dilakukan pada arsitektur domain dimana memiliki banyak node di banyak host.
    
    Untuk dapat mendeploy JDBC JAR file, maka didalam JAR file diperlukan sebuah file bernama `java.sql.Driver` didalam direktori `META-INF/`. Isi file tersebut adalah fully qualified class name dari JDBC driver tersebut, misalnya untuk PostgreSQL adalah `org.postgresql.Driver` sedangkan untuk Oracle DB adalah `oracle.jdbc.OracleDriver`

    >> Biasanya sebuah JDBC driver type-4, yang biasa digunakan sudah memiliki file tersebut, sehingga sudah bisa langsung kita deploy ke JBoss EAP tanpa perlu perubahan.

 2. Membuat JDBC sebagai module di direktori `module/`. Contoh struktur direktori sebuah module JDBC driver adalah seperti berikut:

	```
	modules
	│
	├── postgresql
       └── main
           ├── module.xml
           └── postgresql-9.3-1102.jdbc4.jar
    ```

Ada beberapa cara untuk membuat atau mengkonfigurasi datasource di JBoss EAP, yaitu

1. Cara manual dengan membuat module JDBC driver dan mengubah file konfigurasi XML.
2. Dengan JBoss Command Line Interface (CLI). 
3. Menggunakan web management console

## Pastikan mesin JBoss EAP dapat terkoneksi ke mesin Database

Sebelum menkonfigurasi sebaiknya dicek dulu kesiapan mesin Database dan jaringan antar mesin EAP dan mesin Database jika
memang keduanya jalan di mesin yang berbeda.

- Gunakan perintah `ping` untuk cek koneksi network.
- Gunakan perintah `telnet` untuk cek koneksi ke port service Database. Untuk PostgreSQL, biasanya port yang digunakan 
  (default) adalah `5432` 

	```
   	telnet DATABASE_HOSTNAME 5432
   	```

## Buat Module JDBC Driver di JBoss EAP

Download PostgreSQL JDBC driver dari https://jdbc.postgresql.org/download.html
Jika kita menggunakan Java 1.7 atau maka download file driver JDBC41, sedangkan jika JRE yang digunakan adalah versi 1.6
download file driver JDBC4.

1. Buat direktori `<EAP_INSTALL_DIR>/modules/postgresql/main` untuk menyimpan file JDBC library yang akan di-load oleh EAP sebagai module. 

	```
	cd <EAP_INSTALL_DIR>/modules
	mkdir postgresql/main
	```

2. Simpan file JDBC driver, misalnya `postgresql-9.3-1102.jdbc4.jar` di folder tersebut.
   Buat file `module.xml` di folder tersebut yang isinya seperti ini:

	```xml
	<?xml version="1.0" encoding="UTF-8"?>  
	<module xmlns="urn:jboss:module:1.0" name="org.postgresql">  
	 <resources>  
	 <resource-root path="postgresql-9.3-1102.jdbc4.jar"/>  
	 </resources>  
	 <dependencies>  
	 <module name="javax.api"/>  
	 <module name="javax.transaction.api"/>  
	 </dependencies>  
	</module>
	```
	
   Karakter pertama pada file tersebut tidak boleh spasi atau karakter lainnya tapi harus dimulai dengan `<?xml`

   Selai kita membuat module untuk JDBC driver.

## Buat Konfigurasi Datasource

3. Sekarang kita buat konfigurasi datasource dengan cara manual. Asumsi anda akan menjalankan Jboss EAP dalam mode standalone, maka ubah file konfigurasi EAP yang digunakan (`standalone.xml`). 
   
    Pada elemen `datasources` ubah konfigurasi datasource dengan `pool-name` ExampleDS menjadi seperti berikut:

	```xml
	<datasource jndi-name="java:jboss/datasources/ExampleDS" pool-name="ExampleDS" enabled="true" 
	            use-java-context="true" spy="true">
	    <connection-url>jdbc:postgresql://localhost:5432/bpms</connection-url>
	    <driver-class>org.postgresql.Driver</driver-class>
	    <driver>postgresql</driver>
	    <security>
	        <user-name>postgres</user-name>
	        <password>password</password>
	    </security>
	</datasource> 
	```

    Tambahkan elemen dibawahnya

	```xml
	<drivers>
	    <driver name="postgresql" module="org.postgresql">
	        <xa-datasource-class>org.postgresql.xa.PGXADataSource</xa-datasource-class>
	    </driver>
	    ...
	</drivers>
	```

    Selain cara manual diatas, kita juga bisa membuat datasource dari Web Management Console, dari menu 
    Configuration > Connection > Datasource.

4. Start atau restart JBoss EAP 
   
    Jika JBoss EAP belum dijalankan, maka anda bisa menjalankannya dengan perintah `standalone.sh` (Linux)
    dari direktori `<EAP_INSTALL_DIR>/bin/`


    Jika JBoss EAP sudah jalan maka anda bisa menstopnya dengan perintah `./jboss-cli.sh -c command=:shutdown`
    atau jika JBoss EAP jalan di port yang lain `./jboss-cli.sh -c controller=HOSTNAME:PORT command=:shutdown` 
    

## Test JDBC driver 

7. Untuk melihat apakah driver sudah benar dikonfigurasi kita bisa cek dengan perintah berikut. Pertama jalankan 
    jboss command-line:

    ```
    cd <EAP_INSTALL_DIR>/bin
    ./jboss-cli.sh
    ```
    
    Setelah masuk ke command prompt seperti ini `[disconnected /]` 
    
    ```
    connect localhost
    /subsystem=datasources:installed-drivers-list
    
    ```
    
    Jika perintah `connect` diatas tidal berhasil dan muncul error `ERROR: The controller is not available at 127.0.0.1:9999`, 
     coba cek port management JBoss EAP mungkin tidak menggunakan port yang standar.
    
    Hasil dari perintah tersebut seperti ini:
    

    ```json
    {
	    "outcome" => "success",
	    "result" => [
	        {
	            "driver-name" => "h2",
	            "deployment-name" => undefined,
	            "driver-module-name" => "com.h2database.h2",
	            "module-slot" => "main",
	            "driver-datasource-class-name" => "",
	            "driver-xa-datasource-class-name" => "org.h2.jdbcx.JdbcDataSource",
	            "driver-class-name" => "org.h2.Driver",
	            "driver-major-version" => 1,
	            "driver-minor-version" => 3,
	            "jdbc-compliant" => true
	        },
	        {
	            "driver-name" => "postgresql",
	            "deployment-name" => undefined,
	            "driver-module-name" => "org.postgresql",
	            "module-slot" => "main",
	            "driver-datasource-class-name" => "",
	            "driver-xa-datasource-class-name" => "org.postgresql.xa.PGXADataSource",
	            "driver-class-name" => "org.postgresql.Driver",
	            "driver-major-version" => 9,
	            "driver-minor-version" => 3,
	            "jdbc-compliant" => false
	        }
	    ]
	}
    ```
    
8. Test koneksi dengan menggunakan perintah berikut dari JBoss CLI prompt:

    ```
    /subsystem=datasources/data-source=ExampleDS:test-connection-in-pool
    ```
    
    ```json
    { 
       "outcome" => "success", 
       "result" => [true] 
    }
    ```
    
## Cara membuat DataSource dan JDBC driver module dengan command-line

Berikut cara membuat driver module dan datasource dengan menggunakan CLI. Kali ini kita akan buat datasource untuk koneksi ke database MySQL.

1.  Download [MySQL DJBC driver (Connector/J)](http://dev.mysql.com/downloads/connector/j/). Pilih 'Platform Independent' driver, download ZIP atau tar.gz file, kemudian ekstrak dan copy file `mysql-connector-java-X.X.X-bin.jar` ke direktori `/tmp`

2.  Pastikan JBoss EAP sudah jalan dalam mode standalone. Setelah masuk dan terkonsi ke EAP dengan menggunakan JBoss CLI lakukan perintah berikut untuk membuat module JDBC: 

	```
	[standalone@localhost:9999 /] module add --name=com.mysql --resources=/tmp/mysql-connector-java-5.0.8-bin.jar --dependencies=javax.api,javax.transaction.api
	```
	
	Perintah tersebut akan membuat direktori `<EAP_INSTALL_DIR>/modules/org/mysql/main/`, meng-copy file JAR ke direktori tersebut dan  secara otomatis membuat file `module.xml`

3. Restart JBoss EAP dengan perintah berikut pada CLI

	```
	[standalone@localhost:9999 /] :shutdown(restart=true)
	```

4.  Buat file text dengan nama `create-mysql-ds.cli` yang isinya sebagai berikut:

	```
	connect
	batch
	
	/subsystem=datasources/jdbc-driver=mysql:add( \
	driver-module-name=org.mysql, \
	driver-name=mysql, \
	driver-class-name=com.mysql.jdbc.Driver)
	 
	/subsystem=datasources/data-source=MySQL_DS:add( \
	jndi-name=java:jboss/datasources/MySQL_DS, \
	driver-name=mysql, \
	connection-url=jdbc:mysql://localhost:3306/dbdev, \
	user-name=root, \
	password=admin)
	
	run-batch
	:reload
	```

	Kemudian jalankan dengan perintah
	
	```
	jboss-cli.sh --file=create-mysql-ds.cli
	```
 
5.  Anda bisa test dengan cara yang sama dengan cara sebelumnya dan bisa cek konfigurasi file XML dari EAP yang sudah berubah, mirip seperti saat kita buat datasource manual atau lewat Web Management Console.

	- Login ke Web Mangement Console, navigasikan ke menu Configuration, klik Connecor > Datasources
	- Klik MySQL_DS, kemudian klik tab Connection lalu klik tombol "Test Connection"

	
## Mendeploy JDBC driver dan membuat Datasource dari Web Management Console.

JDBC driver dapat dideploy ke JBoss EAP sebagaimana aplikasi WAR atau EAR untuk kemudian digunakan oleh datasource. Seperti
yang telah disebutkan diawal...

"""
Cara ini lebih direkomendasikan karena lebih mudah dan dapat dengan mudah dilakukan pada arsitektur domain dimana memiliki
banyak node di banyak host.
    
Untuk dapat mendeploy JDBC JAR file, maka didalam JAR file diperlukan sebuah file bernama `java.sql.Driver` didalam direktori
`META-INF/`. Isi file tersebut adalah fully qualified class name dari JDBC driver tersebut, misalnya untuk PostgreSQL adalah
`org.postgresql.Driver` sedangkan untuk Oracle DB adalah `oracle.jdbc.OracleDriver`
"""

1.  Download MariaDB Java Connector (JDBC driver) dari link berikut

	[mariadb-java-client-1.1.8.jar](https://downloads.mariadb.org/interstitial/client-java-1.1.8/mariadb-java-client-1.1.8.jar?serve)

2.  Check apakah driver sudah memiliki file `META-INF/services/java.sql.Driver` dengan perintah berikut:

	```
	jar tf ~/Downloads/mariadb-java-client-1.1.8.jar | grep META-INF
	```
3. Jika belum ada anda bisa membuat direktori `META-INF/services/` di direktori saat ini anda berada. Kemudian buat file text `java.sql.Driver` dengan isi satu baris berikut

	```
	org.mariadb.jdbc.Driver
	```
	
	Setelah itu anda bisa meng0-update JAR file tersebut dengan perintah (anda harus di direktori dimana META-INF berada)
	
	```
	jar uf mariadb-java-client-1.1.8.jar META-INF/services/java.sql.Driver
	```
	
	Anda bisa cek lagi untuk memastikan file tersebut sudah ada di JAR file.

4. Deploy file JAR tersebut dari Web Management Console. Login ke Managemen Console, navigasikan ke menu Deployment, lalu klik tombol "Add", klik Browse, pilih file-nya, klik "Enable" kemudian Save.

	Anda akan lihat di EAP console, log seperti ini yang menunjukan file sudah berhasil di-deploy
	
	```
	15:42:21,561 INFO  [org.jboss.as.repository] (HttpManagementService-threads - 11) JBAS014900: Content added at location /home/jboss/EAP-6.4/standalone/data/content/84/f5b706de4dfcb0915eca8e4fd67fcf796ae868/content
	15:42:21,896 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-3) JBAS015876: Starting deployment of "mariadb-java-client-1.1.8.jar" (runtime-name: "mariadb-java-client-1.1.8.jar")
	```
5.  Navigasi ke menu COnfiguration, lalu klik Connector > Datasources, lalu klik tombol "Add", sebuah wizard window akan muncul. Masukan informasi berikut:

		Name: mariadb_DS
		JNDI Name: java:jboss/datasource/mariadb_DS

	- Klik `mariadb-java-client-1.1.8.jar` pada daftar "Detected Driver"
	- Masukan Connection URL, Username dan Password, klik Test Connection kemudian Done
	- Anda akan melihat "mariadb_DS" ada di daftar JDBC Datasources
	- Pilih mariadb_DS tersebut kemudian klik "Enable" 

6.  Eksplorasi fields yang bisa diatur untuk sebuah datasource.
   
    Klik tab Attibutes, Connection, Pool, Validation dan Timeouts.
    
    Pada masing-masing tab tersebut, klik link "Need Help?" untuk membaca pejelasan dari masing-masing field yang bisa diatur.

	>> QUIZ: Karena sangat pentingnya fields	tersebut untuk tuning JBoss EAP, coba jawab apa fungsi beberapa fields yang penting berikut:
	- share-prepared-statements
	- prepared-statement-cache-size
	- use-ccm
	- min-pool-size
	- max-pool-size
	- prefill (untuk pool)
	- set-tx-query-timeout
	- query-timeout
	- blocking-timeout-millis
	- idle-timeout-minutes
	- allocation-retry
	- allocation-retry-wait-millis
	
