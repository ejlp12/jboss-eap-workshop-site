Jika kita menggunakan Web Server, secara default biasanya Web Server akan memberikan log dari setiap request/response HTTP di sebuah file yang kita kenal dengan nama access log.

Setiap request HTTP ke JBoss EAP juga dapat di-log ke suatu file, dengan cara menambahkan konfigurasi `access-log` elemen di dalam elemen `virtual-server` seperti berikut: 

```xml
        <subsystem xmlns="urn:jboss:domain:web:2.2" default-virtual-server="default-host" native="false">
            <connector name="http" protocol="HTTP/1.1" scheme="http" socket-binding="http"/>
            <virtual-server name="default-host" enable-welcome-root="true">
                <alias name="localhost"/>
                <alias name="example.com"/>
                <access-log pattern='%h %l %u %t %r %s %b %{Referer}i %{User-Agent}i %S %T'>
                    <directory path="./" relative-to="jboss.server.log.dir" />
                </access-log>
            </virtual-server>
            ...
        </subsystem>
```

Contoh output di file `access_log.2015-06-25` di direktori `<JBOSS_HOME>/standalone/log/`
```
0:0:0:0:0:0:0:1 - - [25/Jun/2015:07:13:16 +0700] GET /SimpleJspServletDB/ HTTP/1.1 200 321 - Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/43.0.2357.124 Safari/537.36 xTENn2grp3JWXuIO+YXZtzHH 0.155
```

Format yang ditampilkan di access log menggunakan mengikuti pattern yang kita spesifikasikan di konfigurasi diatas, variable yang dapat ditampilkan adalah sebagai berikut:

> Untuk lebih detail lihat [lihat dokumentasi](http://docs.jboss.org/jbossweb/7.0.x/config/valve.html)

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

Selain itu informasi dari cookie, incoming header,  Session atau informasi lain dari ServletRequest  juga dapat ditampilkan dengan format seperti berikut:

%{xxx}i for incoming headers
%{xxx}o for outgoing response headers
%{xxx}c for a specific cookie
%{xxx}r xxx is an attribute in the ServletRequest
%{xxx}s xxx is an attribute in the HttpSession



Lebih detail bisa dilihat di [How to enable access logging for JBoss EAP 6](https://access.redhat.com/solutions/185383)
