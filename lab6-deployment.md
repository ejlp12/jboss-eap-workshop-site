# LAB Deployment Aplikasi

Men-deploy aplikasi menggunakan Admin Console
---------------------------------------------

1.  Pastikan JBoss EAP dijalankan dalam mode Standalone dengan konfigurasi default port
2.  Akses [Admin Console](http://localhost:8080) dan login menggunakan user admin
   
3.  Klik menu "Deployment"
4.  Klik tombol "Add", dialog window "Create Deployment" akan muncul, kita bias memilih
	- Managed
	- Unmanaged
5.  Pilih __Managed__ anda klik tombol "Choose File" untuk memilih file aplikasi (JAR, WAR atau EAR) lalu browse lokasi file `jboss-numberguess.war` lalu klik "Open". Klik "Next"

	>> Aplikasi `numberguess` adalah aplikasi yang ada di QuickStart yang mendemonstrasikan penggunaan *CDI 1.0*  (Contexts and Dependency Injection) dan *JSF 2.1* (JavaServer Faces). Source code aplikasi tersedia di direktori `<EAP_INSTALL_DIR>/jboss-eap-6.4.0.GA-quickstarts/numberguess/src/`

6.  Biarkan Name dan Runtime Name seperti apa adanya. Biarkan opsi  "Enabled" tidak dipilih (unchecked), klik "Save"

	>> Dengan pilihan enable/disable aplikasi, kita akan dapat melakukan upload aplikasi dan menentukan kapan aplikasi akan dijakankan
	
	Pada console atau `server.log` kita akan lihat output seperti ini
	
	```
	06:00:54,733 INFO  [org.jboss.as.repository] (HttpManagementService-threads - 7) JBAS014900: Content added at location /Users/eariobow/Servers/EAP-6.4.0/standalone/data/content/7c/3afcbee2ad13fbc90a17f37ea27a6d48da5378/content	
	```

7.  Klik `jboss-numberguess.war` di list Deployment, lalu klik tombol "En/Disable" untuk membuat aplikasi tersebut dijakankan (enable), klik tombol Confirm pada dialog windows   

	```
06:22:49,351 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-8) JBAS015876: Starting deployment of "jboss-numberguess.war" (runtime-name: "jboss-numberguess.war")
06:22:49,392 INFO  [org.jboss.weld.deployer] (MSC service thread 1-7) JBAS016002: Processing weld deployment jboss-numberguess.war
06:22:49,404 INFO  [org.jboss.weld.deployer] (MSC service thread 1-7) JBAS016005: Starting Services for CDI deployment: jboss-numberguess.war
06:22:49,405 INFO  [org.jboss.weld.deployer] (MSC service thread 1-3) JBAS016008: Starting weld service for deployment jboss-numberguess.war
06:22:49,423 INFO  [org.jboss.web] (ServerService Thread Pool -- 108) JBAS018210: Register web context: /jboss-numberguess
06:22:49,437 INFO  [javax.enterprise.resource.webcontainer.jsf.config] (ServerService Thread Pool -- 108) Initializing Mojarra 2.1.28-jbossorg-6  for context '/jboss-numberguess'
06:22:49,507 INFO  [org.jboss.as.server] (HttpManagementService-threads - 17) JBAS015859: Deployed "jboss-numberguess.war" (runtime-name : "jboss-numberguess.war")
	```
	
8. Akses aplikasi helloworld dari browser [http://localhost:8080/jboss-numberguess](http://localhost:8080/jboss-numberguess)

9. Delete aplikasi dengan mengklik `jboss-numberguess.war` di list Deployment, lalu klik tombol "Remove", lalu klik Confirm


Men-deploy aplikasi secara manual 
---------------------------------

1.  Pastikan JBoss EAP dijalankan dalam mode Standalone dengan konfigurasi default port
2.  Pastikan Deployment Scanner di-enable. Login ke Management Console, pilih menu "Configuration" lalu expand menu "Core" yang ada di bagian sebelah kiri. Anda bisa mengubah setting Deployment Scanner dengan mengklik tombol "Edit"
3.  Ubah Auto-deploy Zipped menjadi disabled dengan mengklik (uncheck), kemudian klik tombol Save
4.  Copy file `server\target\jboss-shopping-cart-server.jar` ke direktori `<EAP_INSTALL_DIR>/standalone/deployment`

	Di console anda akan melihat output seperti ini
	```
	06:46:23,327 INFO  [org.jboss.as.server.deployment.scanner] (DeploymentScanner-threads - 1) JBAS015003: Found jboss-shopping-cart-server.jar in deployment directory. To trigger deployment create a file called jboss-shopping-cart-server.jar.dodeploy	
	```

5.  Untuk membuat file `jboss-shopping-cart-server.jar` di-deploy, kita perlu membuat file dengan nama yang sama ditambah akhiran (postfix) `.dodeploy`. Di Linux anda bisa membuat file kosong tersebut dengan perintah

	```
	touch jboss-shopping-cart-server.jar.dodeploy
	```
	
	Setelah file .dodeploy dibuat, deployment scanner akan mulai mendeploy aplikasi tersebut dan di console kita bisa lihat output sepert ini:

	```
	06:38:27,071 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-4) JBAS015876: Starting deployment of "jboss-shopping-cart-server.jar" (runtime-name: "jboss-shopping-cart-server.jar")
	06:38:27,192 INFO  [org.jboss.as.ejb3.deployment.processors.EjbJndiBindingsDeploymentUnitProcessor] (MSC service thread 1-3) JNDI bindings for session bean named ShoppingCartBean in deployment unit deployment "jboss-shopping-cart-server.jar" are as follows:

		java:global/jboss-shopping-cart-server/ShoppingCartBean!org.jboss.as.quickstarts.sfsb.ShoppingCart
		java:app/jboss-shopping-cart-server/ShoppingCartBean!org.jboss.as.quickstarts.sfsb.ShoppingCart
		java:module/ShoppingCartBean!org.jboss.as.quickstarts.sfsb.ShoppingCart
		java:jboss/exported/jboss-shopping-cart-server/ShoppingCartBean!org.jboss.as.quickstarts.sfsb.ShoppingCart
		java:global/jboss-shopping-cart-server/ShoppingCartBean
		java:app/jboss-shopping-cart-server/ShoppingCartBean
		java:module/ShoppingCartBean

	06:38:27,323 INFO  [org.jboss.as.server] (DeploymentScanner-threads - 1) JBAS015859: Deployed "jboss-shopping-cart-server.jar" (runtime-name : "jboss-shopping-cart-server.jar")	
	```
4. Lihat direktori `<EAP_INSTALL_DIR>/standalone/deployment`, anda akan melihat file `jboss-shopping-cart-server.jar.deployed` artinya file `jboss-shopping-cart-server.jar` sudah berhasil di-deploy


### Undeploy aplikasi

1. Hapus file `jboss-shopping-cart-server.jar.deployed`
   Deployment scanner akan otomatis melakukan undeployment aplikasi tersebut
   
	```
	6:57:14,505 INFO  [org.jboss.as.server.deployment] (MSC service thread 1-4) JBAS015877: Stopped deployment jboss-shopping-cart-server.jar (runtime-name: jboss-shopping-cart-server.jar) in 10ms
	06:57:14,535 INFO  [org.jboss.as.server] (DeploymentScanner-threads - 1) JBAS015858: Undeployed "jboss-shopping-cart-server.jar" (runtime-name: "jboss-shopping-cart-server.jar")
	```
Mengatur lokasi Root Web
========================

Pada latihan ini, anda akan 'mematikan' halaman default (Welcome file) dari JBoss EAP

1.  Pastikan EAP jalan dalam mode standalone, lalu deploy `jboss-helloworld.war`
2.  Pastikan aplikasi jboss-helloworld sudah jalan dan bisa diakses dari [http://localhost:8080/jboss-helloworld](http://localhost:8080/jboss-helloworld)
3.  Buka file konfigurasi `standalone.xml` lalu cari baris yang miliki elemen `<virtual-server>`, lalu ubah baris tersebut menjadi seperti ini:

	```
	<virtual-server name="default-host" enable-welcome-root="false" default-web-module="jboss-helloworld">
	```

	Set `enable-welcome-root` menjadi __false__ dan set `default-web-module` dengan aplikasi yang sudah di-deploy.

>> Pada versi sebelumnya setting ini tidak bisa dilakukan dari Management Console, tapi di veri 6.4 anda juga dapat melakukan perubahan dari Mgmt Console. Anda bisa mengakses menu "Configuration" > "Web" > "Servlet/HTTP" > __Virtual Servers__, pilih default-host, klik Edit, kemudian ubah "Default Module"
