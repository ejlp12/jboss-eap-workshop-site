
Secara default server JBoss EAP akan menambahkan informasi nama server di HTTP header, seperti ini:

```
HTTP/1.1 200 OK
Server: Apache-Coyote/1.1
Accept-Ranges: bytes
ETag: W/"1496-1427480780000"
Last-Modified: Fri, 27 Mar 2015 18:26:20 GMT
Content-Type: text/html
Content-Length: 1496
Date: Wed, 24 Jun 2015 16:39:31 GMT
```

Untuk mengganti informasi atau value dari header `Server`, kita dapat lakukan dengan cara berikut.

Pada `standalone.xml` tambahkan konfigurasi berikut dibawah element `extensions`:

```
<system-properties>
         <property name="org.apache.coyote.http11.Http11Protocol.SERVER" value="demo-server-name"/>
</system-properties>
```

atau bisa juga dengan cara menjalankan EAP seperti ini:

```
./bin/standalone.sh -Dorg.apache.coyote.http11.Http11Protocol.SERVER=ejlp-server
```

untuk konfigurasi mode domain:

    ```
    <servers>
        <server name="server-one" group="main-server-group" auto-start="true">
        <system-properties>
             <property name="org.apache.coyote.http11.Http11Protocol.SERVER" value="server1"/>
        </system-properties>
        </server>
        <server name="server-two" group="main-server-group" auto-start="true">
                <socket-bindings port-offset="150"/>
        <system-properties>
             <property name="org.apache.coyote.http11.Http11Protocol.SERVER" value="server2"/>
        </system-properties>
        </server>
    </servers>
    ```



![image](https://cloud.githubusercontent.com/assets/3068071/8335520/60a8ee70-1ac8-11e5-9df8-f0fae6f335ac.png)


Info lebih detail: https://access.redhat.com/solutions/21075


