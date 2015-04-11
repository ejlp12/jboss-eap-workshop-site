#LAB5: Memasang Patch pada JBoss EAP

Pada tutorial ini, anda akan mencoba melakukan patching yaitu memasang patch untuk memperbaiki atau memperbarui 
software JBoss EAP yang bersifat minor yaitu biasanya untuk memperbaiki karena adanya bug (kesalahan) software,
untuk menutup lubang/celah keamanan (security fix) atau meningkatkan usability atau performance.

Asumsi saya JBoss EAP 6.3.0 sudah ter-install, dan kita akan coba patch dengan patch 6.3.3 jadi nantinya setelah 
proses patch berhasil JBoss EAP anda menjadi versi 6.3.3. 

Untuk mendapatkan file patch anda harus memiliki (membeli) __[subscription](http://www.redhat.com/en/resources/subscription-guide-red-hat-jboss-middleware)__
Jika anda memiliki active subscription, anda bisa akses [Red Hat Customer Portal](https://access.redhat.com) untuk 
men-download file patch.

Proses patch yang dijelaskan disini dapat dilakukan pada JBoss EAP versi 6.2 keatas.

1. Download file patch `jboss-eap-6.3.3-patch.zip` dari [Download page](https://access.redhat.com/downloads/) Customer Portal.
   Simpan file disebuah direktori, misalnya di `/home/jboss/installer/`
2. Jalankan Command Line Interface (CLI), dengan cara masuk ke direktori `bin` dimana JBoss EAP di-install
   
   ```
   cd /home/jboss/jboss-eap-6.3/bin
   ./jboss-cli.sh
   ```
   Pastikan JBoss EAP sudah dijalankan (started).
   Setelah masuk ke CLI prompt `[disconnected /]` berikan perintah `connect localhost` agar CLI terkoneksi ke JBoss EAP sampai mendapatkan 
   prompt berikut `[standalone@localhost:9999 /]`
   
   ```
   patch apply /home/jboss/installer/jboss-eap-6.3.3-patch.zip
   ```
   
   >>CATATAN: Perintah diatas juga dapat dijalankan dengan satu kali perintah tanpa harus masuk ke prompt CLI dengan
     perintah `$JBOSS_HOME/bin/jboss-cli.sh --command="patch apply /home/jboss/installer//jboss-eap-6.3.3-patch.zip`
   
   >>CATATAN (lagi): Untuk mode domain anda perlu menmberitahu CLI host yang akan dipatch dengan opsi `--host`. 
     Jadi perintahnya seperti ini
     ```
     [domain@localhost:9999 /] patch --host=master apply /NotBackedUp/jboss-eap-6.2.1.zip
     {
       "outcome" : "success",
       "response-headers" : {
          "operation-requires-restart" : true,
          "process-state" : "restart-required"
       },
       "result" : null,
      "server-groups" : null
     }
     ```
3. Cek hasil patching dengan perintah `patch history` pada CLI prompt
   Contoh output perintah tersebut adalah seperti berikut:
   
   ```
   {              
    "outcome" : "success",
    "result" : [
        {
            "patch-id" : "jboss-eap-6.3.3.CP",
            "type" : "cumulative",
            "applied-at" : "4/10/15 9:36 PM"
        },
        {
            "patch-id" : "jboss-eap-6.3.2.CP",
            "type" : "cumulative",
            "applied-at" : "4/10/15 9:36 PM"
        },
        {
            "patch-id" : "jboss-eap-6.3.1.CP",
            "type" : "cumulative",
            "applied-at" : "4/10/15 9:36 PM"
        }
    ],
    "server-groups" : null,
    "response-headers" : {"process-state" : "restart-required"}
   }
  ```
  
  4. Restart JBoss EAP
   
   
