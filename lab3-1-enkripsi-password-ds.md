## Mengenkripsi Password Database yang digunakan pada Data Source

Untuk lebih meningkatkan keamanan dari JBoss EAP, setiap password yang disimpan di file konfigurasi sebaiknya diset tidak dalam bentuk plain text tapi disimpan
dalam bentuk terenkripsi.

Untuk membuat password terenkripsi lakukan langkah berikut:

1.  Enkripsi password dengan menggunakan perintah berikut:

    > Berikut ini adalah perintah di Linux (Bash shell):
    
    ```
    export JBOSS_HOME=/home/jboss/jboss-eap-6.1
    export CLASSPATH=$JBOSS_HOME/modules/system/layers/base/org/picketbox/main/picketbox-4.0.17.SP2-redhat-2.jar:$JBOSS_HOME//modules/system/layers/base/org/jboss/logging/main/jboss-logging-3.1.2.GA-redhat-1.jar:$CLASSPATH

    java  org.picketbox.datasource.security.SecureIdentityLoginModule <password>
    ```

    Ganti `<password>` dengan password yang akan digunakan untuk mengakses database.

    Hasil dari perintah tersebut adalah sebagai berikut: `Encoded password: 9fdd42c2a7390d3`
    
    Copy & paste output dari password yang sudah terenkripsi, dalam hal ini "9fdd42c2a7390d3" untuk kemudian di letakan di konfigurasi `standalone.xml` seperti dibawah ini.

2.  Tambahkan konfigurasi berikut didalam elemen `security-domains` di `standalone.xml`:

    ```
    <security-domain name="encrypted-ds" cache-type="default">  
      <authentication>  
        <login-module code="org.picketbox.datasource.security.SecureIdentityLoginModule" flag="required">  
          <module-option name="username" value="dbUserName"/>  
          <module-option name="password" value="9fdd42c2a7390d3"/>  
          <module-option name="managedConnectionFactoryName" value="jboss.jca:service=LocalTxCM,name=NamaDataSource_ConnectionPool"/>  
        </login-module>  
      </authentication>  
    </security-domain>
    ```

    Perhatikan `NamaDataSource_ConnectionPool` harus sesuai dengan nama datasource pool-name yang akan menggunakan login-modul tersebut


3.  Didalam element `datasource`, hapus bagian ini

    ```
     <datasource jndi-name="java:/blahblah" pool-name="NamaDataSource_ConnectionPool" ... >
        ...
         <!-- Hapus atau beri tanda komentar seperti ini
         <security> 
            <user-name>sa</user-name> 
            <password>sa</password> 
         </security>
         -->
    ```
    
    Ganti menjadi seperti ini:
    
    ```
    <datasource jndi-name="java:/blahblah" pool-name="NamaDataSource_ConnectionPool" ... >
    ...
        <security> 
            <security-domain>encrypted-ds</security-domain>
        </security>
    </datasource>
    
    Perhatikan value dalam element `security-domain` tersebut adalah `encrypted-ds` yaitu value dari attribute name dari element `security-domain` diatas.
    ```

