## Menangani Database Connection Leak

Koneksi yang tidak di-close oleh programmer akan membuat koneksi menggantung di connection pool sehingga connection terlihat seperti masih digunakan dan tidak reusable. Saat kondisi 
seperti ini banyak terjadi, bisa jadi semua connection object yang ada di connection pool tidak ada lagi yang available. Hal ini mengakibatkan aplikasi tidak memperoleh objek koneksi. Biasanya application server akan menunggu untuk waktu tertentu, jika sampai batas waktu tersebut habis maka akan terjadi error seperti ini:

```
javax.resource.ResourceException: IJ000655: No managed connections available within configured blocking timeout (30000 [ms])
```

JBoss EAP dapat secara otomatis mendeteksi dan menutup (close) object ResultSet, Statement atau Connection yang menggantung (leak connection) dengan menggunakan Cached Connection Manager (CCM). Untuk menggunakan CCM, ikut langkah berikut:

1. Set datasource agar menggunakan Cached Connection Manager (CCM) dengan menambahkan attribute `use-ccm="true"` seperti dibawah ini.


	```
	<datasource jndi-name="..." pool-name="..." enabled="true" use-ccm="true">
	```

2. Tambahkan konfigurasi logging pada elemen `<subsystem xmlns="urn:jboss:domain:logging:1.4">` agar kita bisa melihat apa yang terjadi saat koneksi tidak di-close()

	```
	<subsystem xmlns="urn:jboss:domain:jca:1.1">
	  ...
	  <cached-connection-manager debug=true error=true/>
	</subsystem> 
	```


Cara diatas juga dapat dilakukan dengan menggunakan JBoss CLI:

```
/subsystem=jca/cached-connection-manager=cached-connection-manager/:write-attribute(name=error,value=true)

/subsystem=jca/cached-connection-manager=cached-connection-manager/:write-attribute(name=debug,value=true)
```

Hasil output jika terjadi kasus object resultset, statement atau connection yang tidak di-close, kemudian di-close otomatis oleh CCM adalah seperti ini:

```
10:52:08,983 WARN  [org.jboss.jca.adapters.jdbc.WrappedConnection] (http-/127.0.0.1:8180-1) Closing a result set you left open! Please close it yourself.: java.lang.Throwable: STACKTRACE
	at org.jboss.jca.adapters.jdbc.WrappedStatement.registerResultSet(WrappedStatement.java:1357)
	at org.jboss.jca.adapters.jdbc.WrappedStatement.executeQuery(WrappedStatement.java:345)
	at org.apache.jsp.index_jsp._jspService(index_jsp.java:74)
<DIHAPUS>
```

```
10:44:49,602 INFO  [org.jboss.jca.core.api.connectionmanager.ccm.CachedConnectionManager] (http-/127.0.0.1:8180-1) IJ000100: Closing a connection for you. Please close them yourself: org.jboss.jca.adapters.jdbc.jdk6.WrappedConnectionJDK6@1368aae: java.lang.Throwable: STACKTRACE
```

```
10:45:12,808 WARN  [org.jboss.jca.adapters.jdbc.WrappedConnection] (http-/127.0.0.1:8180-1) Closing a statement you left open, please do your own housekeeping: java.lang.Throwable: STACKTRACE
```
