# LAB-4: Mengkonfigurasi JBoss EAP sebagai Service di Linux

Pada LAB ini, kita akan membuat JBoss EAP agar terdaftar sebagai Service di Linux (asumsi anda menggunakan OS Red Hat Enterprise Linux, Fedora atau CentOS)
Jika anda ingin melihat dokumentasi terkait setting JBoss EAP sebagai Service di Linux, anda bisa baca di link berikut:

[4.9.2. Configure JBoss EAP 6 as a Service in Red Hat Enterprise Linux (Zip, Installer)](https://access.redhat.com/documentation/en-US/JBoss_Enterprise_Application_Platform/6.3/html/Installation_Guide/Install_JBoss_Enterprise_Application_Platform_6_Red_Hat_Enterprise_Linux_Service.html)


1. Gunakan script init.d dari bawaan JBoss EAP, biasanya ada di direktori `<EAP_INSTALL_DIR>/bin/init.d/`
   Di direktori tersebut terdapat 3 file:
   ```
   jboss-as.conf - file konfigurasi yang harus disimpan di /etc/jboss-as/
   jboss-as-domain.sh - file start untuk up mode domain yang harus disimpan di /etc/init.d/
   jboss-as-standalone.sh - file start up untuk mode standalone yang harus disimpan di /etc/init.d/
   ```
   
2. Copy file `jboss-as-standalone.sh` di direktori `/etc/init.d`
   Atau buat symbolic link file `jboss-as-standalone.sh` di direktori `/etc/init.d` ke file yang ada di `<EAP_INSTALL_DIR>/bin/init.d/`
   
   Untuk mempermudah cara menjalankan atau menstop JBoss EAP, anda bisa copy atau buat symlink dengan nama yang lebih simple,
   misalnya menjadi `/etc/init.d/jboss-eap/jboss-eap` saja. 
   
   Misal JBoss EAP di-install di direktori `/home/jboss/jboss-eap-6.3/`, jalankan perintah ini dari user root: 
   
   ```
   ln -s /home/jboss/jboss-eap-6.3/bin/init.d/jboss-as-standalone.sh /etc/init.d/jboss-eap/jboss-eap
   ```
   
3. buat symbolic link file `jboss-as.conf` di direktori `/etc/jboss-as`

   ```
   mkdir /etc/jboss-as
   ln -s /home/jboss/jboss-eap-6.3/bin/init.d/jboss-as.conf /etc/jboss-as/jboss-as.conf
   ```

4. Edit file `/etc/jboss-as.conf`

   `gedit /etc/jboss-as.conf`

   Uncomment konfigurasi jadi seperti ini:
   
    ```
    # General configuration for the init.d scripts,
    # not necessarily for JBoss AS itself.
    
    # The username who should own the process.
    JBOSS_USER=jboss-as

    # The amount of time to wait for startup
    STARTUP_WAIT=30
    
    # The amount of time to wait for shutdown
    SHUTDOWN_WAIT=30

    # Location to keep the console log
    JBOSS_CONSOLE_LOG=/var/log/jboss-as/console.log
    ```

5. Tambahkan user `jboss-as`, dan direktori untuk log serta untuk menyimpan pid file. Jalankan command ini dari user `root`

   ```
   adduser jboss-as
   
   mkdir /var/log/jboss-as
   chown jboss-as /var/log/jboss-as
   chmod 755 /var/log/jboss-as
   
   mkdir /var/run/jboss-as
   chown jboss-as /var/run/jboss-as
   chmod 755 /var/run/jboss-as
   ```

6. Script `init.d` tersebut menggunakan asumsi direktori instalasi JBoss EAP  di `/usr/share/jboss-as` 
   Kita dapat membuat agar saat login user `jboss-as` memiliki environment variable $JBOSS_HOME dengan value direktori tempat JBoss EAP di-install.
   Untuk mempermudah, kita buat link saja dari `/usr/share/jboss-as` ke direktori tempat JBoss EAP di-install misalnya di 
   `/home/jboss/jboss-eap-6.3`

   ```
   ln -s /home/jboss/jboss-eap-6.3 /usr/share/jboss-as
   ```
   
7. Register srcipt di `init.d` sebagai Linux service

   ```
   chkconfig --add  jboss-eap
   chkconfig  jboss-eap --level 2345 on 
   ```

8. Test jalankan JBoss EAP dan tunggu sampai muncul output [OK] atau [FAILED]
   
   ```
   service jboss-eap start
   ```
   
9. Jika saat start diatas hasilnya [FAILED], coba cek log atau console output di `/var/log/jboss-as/console.log`
   
   ```
   cat /var/log/jboss-as/console.log |less
   ```
   
    
