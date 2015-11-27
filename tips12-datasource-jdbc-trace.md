## Memonitor JDBC

Adakalanya kita butuh melakukan trace di level JDBC, untuk melihat query SQL apa yang dijalankan aplikasi. 
Berikut caranya:

1. Menambahkan attribute __`spy="true"`__ di datasource element di file konfigurasi (`standalone.xml`)

	```
	<datasource jndi-name="..." pool-name="..." enabled="true" spy="true">
	```
	
	atau dengan JBoss CLI:
	```
	/subsystem=datasources/data-source=DS_POOL_NAME/:write-attribute(name=spy,value=true)
	```

2. Menambahkan kategori `jboss.jdbc.spy` di subsystem logging

	```
	<logger category="jboss.jdbc.spy">
	   <level name="TRACE"/>
	</logger>
	```
	
	atau dengan JBoss CLI:
	```
	/subsystem=logging/logger=jboss.jdbc.spy/:add(level=TRACE)
	```
	
3. Setelah melakukan perubahan tersebut lakukan restart

4. Hasil dari 

```
tail -f <EAP_INSTALL_DIR>/standalone/logs/server.log | grep jboss.jdbc.spy
```

```
10:45:18,276 DEBUG [jboss.jdbc.spy] (http-/127.0.0.1:8180-1) java:jboss/datasources/ExampleDS [DataSource] getConnection()
10:45:18,277 DEBUG [jboss.jdbc.spy] (http-/127.0.0.1:8180-1) java:jboss/datasources/ExampleDS [Connection] createStatement()
10:45:18,277 DEBUG [jboss.jdbc.spy] (http-/127.0.0.1:8180-1) java:jboss/datasources/ExampleDS [Statement] setQueryTimeout(120)
10:45:18,277 DEBUG [jboss.jdbc.spy] (http-/127.0.0.1:8180-1) java:jboss/datasources/ExampleDS [Statement] executeQuery(SELECT * FROM test)
10:45:18,296 DEBUG [jboss.jdbc.spy] (http-/127.0.0.1:8180-1) java:jboss/datasources/ExampleDS [ResultSet] next()
10:45:18,296 DEBUG [jboss.jdbc.spy] (http-/127.0.0.1:8180-1) java:jboss/datasources/ExampleDS [ResultSet] getString(Name)
10:45:18,296 DEBUG [jboss.jdbc.spy] (http-/127.0.0.1:8180-1) java:jboss/datasources/ExampleDS [ResultSet] getString(num)
10:54:44,467 DEBUG [jboss.jdbc.spy] (http-/127.0.0.1:8180-1) java:jboss/datasources/ExampleDS [ResultSet] close()
10:54:44,468 DEBUG [jboss.jdbc.spy] (http-/127.0.0.1:8180-1) java:jboss/datasources/ExampleDS [Statement] close()
10:54:44,468 DEBUG [jboss.jdbc.spy] (http-/127.0.0.1:8180-1) java:jboss/datasources/ExampleDS [Connection] close()
```

## Memonitor open & close connection di pool

Tambahan: jika anda ingin melihat aktivitas connection pool yaitu saat koneksi dibuka dan ditutup oleh setiap object connection, anda bisa nemambahkan atribut `cached-connection-manager`

```
/subsystem=jca/cached-connection-manager=cached-connection-manager/:write-attribute(name=error,value=true)
``
