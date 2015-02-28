# Mengkonfigurasi Koneksi JBoss EAP ke ProsgreSQL Database

>> Tutorial ini dibuat dengan menggunakan JBoss EAP versi 6.3.0, cara yang sama untuk koneksi ke database lainnya misalnya
MySQL, Oracle Database, MariaDB akan sama saja. Yang membedakan adalah library JDBC, connection URL, driver class, dan 
datasource class.

Sebelum melakukan konfigurasi JBoss EAP agar terkoneksi ke database, pertama tentu perlu dipastikan bahwa database sudah
terinstall dan dapat diakses dari server EAP. 

Untuk dapat terkoneksi ke database, biasanya aplikasi menggunakan datasource yang diakses menggunakan JNDI. Datasource 
tersebut memberikan koneksi database kepada aplikasi. Datasource menggunakan connection-pool yang mengelola koneksi ke
database agar lebih efisien. Tiap objek koneksi yang ada di connection-pool menggunakan JDBC driver untuk membuat koneksi
ke database. Setiap database memiliki JDBC driver-nya masing-masing dan datasource perlu diset agar objek koneksi bisa
dibuat saat dibutuhkan.

Ada beberapa cara untuk membuat atau mengkonfigurasi datasource di JBoss EAP, yaitu

1. Cara manual dengan membuat module JDBC driver dan mengubah file konfigurasi XML.
2. Dengan JBoss Command Line Interface (CLI). 


## Pastikan mesin JBoss EAP dapat terkoneksi ke mesin Database

Sebelum menkonfigurasi sebaiknya dicek dulu kesiapan mesin Database dan jaringan antar mesin EAP dan mesin Database jika
memang keduanya jalan di mesin yang berbeda.

- Gunakan perintah `ping` untuk cek koneksi network.
- Gunakan perintah `telnet` untuk cek koneksi ke port service Database. Untuk PostgreSQL, biasanya port yang digunakan 
  (default) adalah `5432` 

	```
   	telnet DATABASE_HOSTNAME 5432
   	```



## Buat Module DJBC Driver di JBoss EAP

Download PostgreSQL JDBC driver dari https://jdbc.postgresql.org/download.html
Jika kita menggunakan Java 1.7 atau maka download file driver JDBC41, sedangkan jika JRE yang digunakan adalah versi 1.6
download file driver JDBC4.

1. Buat direktori `<EAP_INSTALL_DIR>/modules/system/layers/base/org/postgresql/main` untuk menyimpan file JDBC library yang akan di-load oleh EAP sebagai module. 

	```
	cd <EAP_INSTALL_DIR>/modules/system/layers/base/org/
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

3. Ubah file konfigurasi EAP yang digunakan (`standalone.xml`). 
   
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
    

    ```
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
    
    ```
    { 
       "outcome" => "success", 
       "result" => [true] 
    }
    ```
    
## Cara membuat DataSource dan JDBC driver module dengan command-line

Berikut cara membuat driver module dan datasource dengan menggunakan `jboss-cli.sh`

1. Setelah masuk dan terkonsi ke EAP dengan menggunakan JBoss CLI lakukan perintah berikut untuk membuat module JDBC:

	```
	module add --name=org.mysql --resources=/path/to/mysql-connector-java-5.1.18-bin.jar --dependencies=javax.api,javax.transaction.api
	```

	Pastikan JDBC driver file ada di `/path/to`. Perintah tersebut akan membuat direktori `<EAP_INSTALL_DIR>/modules/org/mysql/main/`, meng-copy file JAR ke direktori tersebut dan  secara otomatis membuat file `module.xml`

2. Buat deklarasi datasource dan driver di file konfigurasi EAP dengan cara memberikan perintah berikut di CLI prompt: 

	``` 
	/subsystem=datasources/jdbc-driver=mysql:add(driver-module-name=org.mysql,driver-name=mysql,driver-class-name=com.mysql.jdbc.Driver)
	 
	/subsystem=datasources/data-source=MySQLDS:add(jndi-name=java:jboss/datasources/MySQLDS, driver-name=mysql, connection-url=jdbc:mysql://localhost:3306/dbdev,user-name=root,password=admin)
	```
3. Ada bisa test dengan cara yang sama dengan cara sebelumnya dan bisa cek konfigurasi file XML dari EAP yang sudah berubah, mirip seperti saat kita buat datasource manual.

