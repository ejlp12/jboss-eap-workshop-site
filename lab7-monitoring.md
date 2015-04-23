# Monitoring JBoss EAP

> Catatan: JBoss EAP yang digunakan adalah versi 6.3. 
  Apa yang ditulis disini mungkin sesuai untuk semua JBoss EAP versi > 6.0 atau JBoss AS versi 7.1 dan WildFly versi 8.X

Untuk memonitor JBoss EAP kita dapat menggunakan beberapa tools diantaranya:
	
- JBoss Command Line Interface (CLI), yaitu dengan perintah `jboss-cli.sh` (Linux) atau `jboss-cli.bat` (Windows)
- Native Management API, yaitu dengan pemrograman menggunakan JBoss remoting
- HTTP Management API (REST)
- JMX Tool, seperti JConsole dan lainnya.
- JBoss Operation Network atau RHQ

# Monitoring Datasource atau Connection Pool

Pada LAB ini kita akan mencoba melakukan pengaturan mode monitoring untuk data source di JBoss EAP, kemudian memonitor menggunakan CLI, HTTP management API, JConsole serta membuat tool sederhana untuk monitoring secara grafis.

## Enable Datasource Statistic 

Untuk memonitor datasource atau koneksi ke database, kita perlu meng-enable statistic dengan cara menambahkan konfigurasi `statistics-enabled="true"` pada elemen datasource yang ingin kita monitor, seperti dibawah ini:


```xml
<datasource jndi-name="..." pool-name="..." enabled="true" use-java-context="true" statistics-enabled="true">
```

Atau bisa juga dengan menggunakan JBoss CLI, untuk mode standalone:

```
[standalone@localhost:9999 /] /subsystem=datasources/data-source=ExampleDS:write-attribute(name=statistics-enabled,value=true)
{
    "outcome" => "success",
    "response-headers" => {
        "operation-requires-reload" => true,
        "process-state" => "reload-required"
    }
}
```

Pada mode domain:

```
[domain@localhost:9999 /] /profile=full/subsystem=datasources/data-source=ExampleDS:write-attribute(name=statistics-enabled,value=true)
{
    "outcome" => "success",
    "result" => undefined,
    "server-groups" => {"main-server-group" => {"host" => {"master" => {
        "server-one" => {"response" => {
            "outcome" => "success",
            "response-headers" => {
                "operation-requires-restart" => true,
                "process-state" => "restart-required"
            }
        }},
        "server-two" => {"response" => {
            "outcome" => "success",
            "response-headers" => {
                "operation-requires-restart" => true,
                "process-state" => "restart-required"
            }
        }}
    }}}}
```

## Datasource & JDBC statistic

Penjelasan mengenai arti dari parameter (metric) yang ditampilkan pada statistic liat di [Datasource Statistics](https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.3/html/Administration_and_Configuration_Guide/Datasource_Statistics.html)

## Memonitor menggunakan JBoss CLI

Berikut cara memonitor menggunakan CLI 

```
/subsystem=datasources/data-source=ExampleDS/statistics=pool:read-resource(recursive=true, include-runtime=true)
```

Untuk JBoss EAP yang jalan dalam mode domain, perintah diatas menjadi seperti ini:

```
/host=master/server=server-one/subsystem=datasources/data-source=ExampleDS/statistics=pool:read-resource(include-runtime=true)
```

## Memonitor dari HTTP Management API

JBoss EAP memiliki management interface yang jalan 

`http://localhost:9990/management/subsystem/datasources/data-source/ExampleDS/statistics/pool?include-runtime=true`

Berikut hasil output dari akses URL tersebut menggunakan browser:

```
{
"ActiveCount": "7",
"AvailableCount": "4",
"AverageBlockingTime": "1",
"AverageCreationTime": "37",
"CreatedCount": "7",
"DestroyedCount": "0",
"InUseCount": "6",
"MaxCreationTime": "253",
"MaxUsedCount": "7",
"MaxWaitCount": "0",
"MaxWaitTime": "1",
"TimedOut": "0",
"TotalBlockingTime": "1",
"TotalCreationTime": "259",
"statistics-enabled": true
}
```

## Memonitor dengan menggunakan JConsole

JConsole adalah JMX graphical-tool bawaan dari distribusi JRE/JDK untuk memonitor JVM atau aplikasi yang mendukung JMX.

Berikut langkah-langkah untuk memonitor menggunakan JConsole:

1. Jalankan JConsole GUI dengan menggunakan perintah `jconsole.sh` (Linux) atau `jconsole.bat` (Windows) yang ada di direktori `<JBOSS_EAP_HOME>/bin/`

2. Pilih __Remote Process__ dan masukan URL `service:jmx:remoting-jmx://HOSTNAME:PORT`, dengan `HOSTNAME` adalah hostname atau IP address dari JBoss EAP server dan PORT adalah port management (native) yang bisa dilihat di file konfigurasi (standalone.xml) seperti dibawah ini:

	```
	<socket-binding name="management-native" interface="management" port="${jboss.management.native.port:9999}"/>
	```

	Jika port-offset tidak diset, berarti port yang harus kita gunakan adalah port 9999.
	
	>> Kita juga dapat menggunakan __Local Process__ jika JConsole yang kita jalankan kita gunakan untuk memonitor atau mengeloala JBoss EAP yang jalan di mesin/host yang sama. Pilih  pada __Local Process__ list suatu proses dengan nama seperti ini `jboss-module.jar -mp /path/...` 

3. Set username/password, lalu klik tombol "Connect"

## Monitoring secara grafis dengan menggunakan Jolokia & Highcart

Dengan [Jolokia](http://www.jolokia.org/) dan [Highchart](http://www.highcharts.com/) kita bisa membuat [monitoring tool sederhana](http://www.nurkiewicz.com/2011/03/jolokia-highcharts-jmx-for-human-beings.html) yang menampilkan grafik perubahan metric yang kita monitor terhadap waktu.


![Contoh grafik yang ditampilkan](https://lh4.googleusercontent.com/-V1pVl5POZ24/TYZJWYKkXNI/AAAAAAAAAaU/vlIn5Dyt1iM/s1600/multiple.png)

Untuk menampilkan grafik monitoring JBoss EAP caranya mudah. Kita akan coba menampilkan monitoring beberapa datasource metric. Ikuti caranya seperti langkah-langkah berikut:

1. Download file WAR dari Jolokia. Saya menggunakan [jolokia-war-1.2.3.war](http://labs.consol.de/maven/repository/org/jolokia/jolokia-war/1.2.3/jolokia-war-1.2.3.war)

2. Masukan file WAR tersebut ke `<EAP_INSTALL_DIR>/standalone/deployment` agar dideploy sebagai aplikasi web di JBoss EAP. Lalu test Jolokia dengan mengakses ke 

	`http://localhost:8180/jolokia-war-1.2.3/read/jboss.as:subsystem=datasources,data-source=ExampleDS,statistics=pool`

	Jika sukses, anda akan mendapatkan response seperti ini:

	```
	{
		"timestamp": 1425082459,
		"status": 200,
		"request": {
			"mbean": "jboss.as:data-source=ExampleDS,statistics=pool,subsystem=datasources",
			"type": "read"
		},
		"value": {
			"AverageBlockingTime": 1,
			"MaxWaitTime": 1,
			"ActiveCount": 7,
			"CreatedCount": 7,
			"MaxCreationTime": 253,
			"MaxWaitCount": 0,
			"TotalCreationTime": 259,
			"AvailableCount": 4,
			"MaxUsedCount": 7,
			"TimedOut": 0,
			"statisticsEnabled": true,
			"TotalBlockingTime": 1,
			"AverageCreationTime": 37,
			"InUseCount": 6,
			"DestroyedCount": 0
		}
	}
	```

3. Download semua file di folder `css`, `js` dan file `index.html` di [repository ini](https://github.com/nurkiewicz/token-bucket/tree/master/src/main/webapp)

    Atau clone [git repository ini](https://github.com/nurkiewicz/token-bucket.git)

4. Copy file `js/index.js` menjadi `js/jboss-datasource`
   Sehingga struktur file yang anda punya menjadi seperti ini:

	```
	├── css
	│   └── tokenbucket.css
	├── index.html
	└── js
	    ├── ext
	    │   ├── highcharts.src.js
	    │   ├── jolokia-simple.js
	    │   └── jolokia.js
	    ├── index.js
	    └── jboss-datasource.js
	```

	
    Edit file `jboss-datasource.js` sehingga bagian atlas, didalam code `factory.create([...])` menjadi seperti ini.
    

    ```
	$(document).ready(function() {
        var factory = new JmxChartsFactory();

        factory.create([
        {
           name: 'jboss.as:data-source=ExampleDS,statistics=pool,subsystem=datasources',
           attribute: 'AvailableCount'
        },
        {
           name: 'jboss.as:data-source=ExampleDS,statistics=pool,subsystem=datasources',
           attribute: 'InUseCount'
        },
        {
           name: 'jboss.as:data-source=ExampleDS,statistics=pool,subsystem=datasources',
           attribute: 'ActiveCount'
        }
        ]);
	});    
    ```
    
    Ubah juga `var jolokia = new Jolokia("/jolokia");` menjadi seperti ini
    
	```
	var jolokia = new Jolokia("http://localhost:8180/jolokia-war-1.2.3");
	```
    
    kemudian ubah type chart dengan menambahkan line `type: 'line'` pada `new Highcharts.Chart(...)` seperti dibawah ini
    

	```
	function createChart(mbeans) {
	      return new Highcharts.Chart({
	              chart: {
	                  renderTo: createNewPortlet(mbeans[0].name),
	                  animation: false,
	                  defaultSeriesType: 'area',
	                  shadow: false,
	                  type: 'line'
	               },
	[DIHAPUS]
	```
	
	Bagian perubahan ini tidak terlalu penting, tapi mungkin ingin diikuti. Saya mengubah 
	
	```
    yAxis: { title: { text: mbeans[0].attribute } },
    ```
    
    menjadi
    

    ```
    yAxis: { title: { text: 'value' } },
    ```

5. Edit file `index.html` dan ubah line berikut 
    
    `<script src="js/index.js"></script>` 
    
    menjadi
    
    `<script src="js/jboss-datasource.js"></script>`

6. Save file `index.html`, kemudian klik dua kali file tersebut agar tampil di browser.

## Monitoring menggunakan JON

Silakan lihat dokumentasi berikut:

https://access.redhat.com/documentation/en-US/Red_Hat_JBoss_Operations_Network/3.3/html/Users_Guide/eap6-chapter.html

https://access.redhat.com/documentation/en-US/Red_Hat_JBoss_Operations_Network/3.3/html/Complete_Resource_Reference/Managed_JBoss.html



