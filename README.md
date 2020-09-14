# HA Proxy

Based on https://hub.docker.com/_/haproxy

### Build container

~~~
$ docker build -t hthunberg/demo-haproxy .
~~~

### Test config file

$ docker run -it --rm --name haproxy-syntax-check hthunberg/demo-haproxy haproxy -c -f /usr/local/etc/haproxy/haproxy.cfg

### HTTP 503 scenario

#### 1: Start HA Proxy and all apache backends

~~~
dco up -d --build && dco logs -f
~~~

Expect the following in the logs when the health check is executed by HA Proxy

~~~
apacheError_1  | 192.168.96.1 - - [14/Sep/2020:10:01:44 +0000] "GET /health HTTP/1.0" 500 26
apache1_1      | 192.168.96.1 - - [14/Sep/2020:10:01:44 +0000] "GET /health HTTP/1.0" 200 490
apache2_1      | 192.168.96.1 - - [14/Sep/2020:10:01:45 +0000] "GET /health HTTP/1.0" 200 490
~~~

#### 2: Start requesting backends

~~~
watch curl -i http://localhost:8888
~~~

Expects the following HTTP response:

~~~
Serving from Apache Server 1
~~~

~~~
Serving from Apache Server 2
~~~

#### 3: Stop one of the backends serving HTTP 200

~~~
dco stop apache1
~~~

Expects the following HTTP response:

~~~
Serving from Apache Server 2
~~~

#### 4: Stop the last of the backends serving HTTP 200

~~~
dco stop apache2
~~~

Expects the following HTTP response:

~~~
<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
~~~

#### 5: Start backends again

~~~
dco start apache1 apache2
~~~

Expects the following HTTP response:

~~~
Serving from Apache Server 1
~~~

~~~
Serving from Apache Server 2
~~~