Jika kita menggunakan Web Server atau Proxy Server, secara default biasanya Web Server akan memberikan log dari setiap request/response HTTP di sebuah file yang kita kenal dengan nama access log.

EAP secara default tidak mengeluarkan access log, jadi jika JBoss EAP server langsung diakses end-users (tidak ada Web Server atau Proxy Server) yang menjadi jembatan sebelum request diterima JBoss EAP maka kita tidak dapat melihat access log. 

Tetapi kita bisa membuat agar EAP dapat mengeluarkan access log yang serupa dengan Apache HTTP server. Jadi setiap request HTTP ke JBoss EAP juga dapat di-log ke suatu file, yaitu dengan cara menambahkan konfigurasi `access-log` elemen di dalam elemen `virtual-server` seperti berikut: 

 Tapi seringkali kita menjalankan 
  Caranya adalah sebagai berikut:

Tambahkan konfigurasi ini di `standalone.xml`

```
<server name="abc" xmlns="urn:jboss:domain:1.3">
...
    <subsystem xmlns="urn:jboss:domain:web:1.2" default-virtual-server="default-host" native="false">
            <connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"/>
            <virtual-server name="default-host" enable-welcome-root="true">
                <alias name="localhost"/>
                <alias name="example.com"/>
               <access-log pattern='%h %l %u %t %r %s %b %{Referer}i %{User-Agent}i %S %T'>
                   <directory path="${jboss.server.name}-accessLog" relative-to="jboss.server.log.dir" />
                </access-log>
            </virtual-server>
        </subsystem>
```
Secara default access log akan disimpan di `${jboss.server.log.dir}/default-host` dimana `default-host` adalah nama yang 
dispesifikasikan di elemen `virtual-server` jika kita tidak menspesifikasikan letak log file dengan elemen `<directory path="">`.

Pada contoh diatas letak dan mana file-nya akan seperti ini `<EAP_INSTALL_DIR>/standalone/log/abc-accessLog/access_log.2015-04-27` 
dengan contoh format lognya adalah seperti ini:

```
127.0.0.1 - - [27/Apr/2015:15:08:09 +0700] GET /queryrunner/theme/style.css HTTP/1.1 404 1116 http://localhost:8080/queryrunner/index.jsp Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36 f5P0X2cu4+zQaUM2i+AXSMRw 0.095
127.0.0.1 - - [27/Apr/2015:15:15:01 +0700] GET /queryrunner/index.jsp HTTP/1.1 200 586 - Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36 vfab6BqFD4608V+vzSL80Fmw 0.066
```

Format yang ditampilkan di access log menggunakan mengikuti __pattern__ yang kita spesifikasikan di konfigurasi diatas, variable yang dapat ditampilkan adalah sebagai berikut:

> Untuk lebih detail untuk membuat pattern bisa dibaca di [dokumentasi](http://docs.jboss.org/jbossweb/7.0.x/config/valve.html)

```
%a - Remote IP address
%A - Local IP address
%b - Bytes sent, excluding HTTP headers, or '-' if zero
%B - Bytes sent, excluding HTTP headers
%h - Remote host name (or IP address if resolveHosts is false)
%H - Request protocol
%l - Remote logical username from identd (always returns '-')
%m - Request method (GET, POST, etc.)
%p - Local port on which this request was received
%q - Query string (prepended with a '?' if it exists)
%r - First line of the request (method and request URI)
%s - HTTP status code of the response
%S - User session ID
%t - Date and time, in Common Log Format
%u - Remote user that was authenticated (if any), else '-'
%U - Requested URL path
%v - Local server name
%D - Time taken to process the request, in millis
%T - Time taken to process the request, in seconds
%I - current request thread name (can compare later with stacktraces)
```

Selain itu informasi dari cookie, incoming header,  Session atau informasi lain dari ServletRequest  juga dapat ditampilkan dengan format seperti berikut:

```
%{xxx}i for incoming headers
%{xxx}o for outgoing response headers
%{xxx}c for a specific cookie
%{xxx}r xxx is an attribute in the ServletRequest
%{xxx}s xxx is an attribute in the HttpSession
```

Lebih detail bisa dilihat di [How to enable access logging for JBoss EAP 6](https://access.redhat.com/solutions/185383)

