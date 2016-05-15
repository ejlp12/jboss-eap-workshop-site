Biasanya kita menginginkan aplikasi web di JBoss EAP untuk bisa diakses dari port 80 (HTTP) atau 443 (HTTPS).

JBoss EAP default-nya menjalanakan protokol HTTP dan HTTPS untuk aplikasi web berturut-turut di port 8080 dan 8443.

JBoss EAP di production environment sebagiknya (best practice) dijalankan oleh user non-root karena alasan keamanan, dan karena alasan keamanan juga user non-root di Linux tidak bisa menjalankan aplikasi yang membuka port dibawah 1024.

Jadi solusinya adalah:

1. Solusi yang paling disarankan adalah menggunakan HTTP Server, atau komponen Load Balancer diluar EAP.
2. Menggunakan port forwarding di Linux (RHEL 6.X)
   
   Login sebagai *root* lalu jalankan beberapa perintah berikut
   
   ```
  sysctl -w net.ipv4.ip_forward=1
  iptables -F -t nat
  iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to :8080
  iptables -t nat -A OUTPUT -o lo -p tcp --dport 80 -j REDIRECT --to-ports 8080
   ```
   
   Edit file `/etc/sysconfig/iptables-config`
   
   ```
   IPTABLES_SAVE_ON_RESTART="yes"
   IPTABLES_SAVE_ON_STOP="yes"
   ``` 
   
   Save konfigurasi iptables diatas dan pastikan iptables selalu enable saat Linux start
   
   ```
   service iptables save
   chkconfig iptables on
   ```
   
   
