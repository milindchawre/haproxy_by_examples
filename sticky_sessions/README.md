# Sticky sessions in haproxy

**Sticky Sessions**: The problem with using a load balancer is that by default, traffic bounces from one server to the next. Some applications require that a user continues to connect to the same backend server. This persistence is achieved through sticky sessions.

#### Pre-requisite
- A Virtual Machine(VM) running any OS centos, ubuntu, etc.
- [Docker] installed on the VM.
- Ports 7979-8082 opened in the security group.

#### Steps to enable sticky sessions in HAProxy
- First we create docker image for our 2 apache webserver using these commands.
```sh
docker build -t apache1 -f Dockerfile1 .
docker build -t apache2 -f Dockerfile2 .
```
- Now we create docker image for HAProxy.
```sh
docker build -t haproxy:sticky -f Dockerfile3 .
```
- Now we run our apache webserver.
```sh
docker run -d -it -p 8081:80 apache1
docker run -d -it -p 8082:80 apache2
```
- Verify the webserver is running by hitting this URL in browser. URLs `http://<machine-ip>:8081` and `http://<machine-ip>:8082` The browser should show `server-1` or `server-2` as an output.
- Now run HAProxy.
```sh
docker run -d -it -e machine_ip=<ip-of-machine> -p 7979:7979 -p 8080:80 haproxy:sticky
```
- Now hit port 8080 where HAProxy is load balancing the webservers. URL `http://<machine-ip>:8080`. The browser should now output either `server-1` and `server-2` and stick to that server. In my case its sticking to `server-2`.
![alt text](https://raw.githubusercontent.com/milindchawre/haproxy_by_examples/master/sticky_sessions/img/haproxy_sticky1.png)
![alt text](https://raw.githubusercontent.com/milindchawre/haproxy_by_examples/master/sticky_sessions/img/haproxy_sticky2.png)

#### How it works
- The sticky sessions is achieved by the following configuration in haproxy.cfg file
```sh
backend apache_backend
   balance roundrobin
   cookie SERVERID insert indirect nocache
   server web1 ${machine_ip}:8081 check cookie w1
   server web2 ${machine_ip}:8082 check cookie w2
```
- the line `cookie SERVERID insert indirect nocache` tells HAProxy to setup a cookie called SERVERID only if the user did not come with such cookie. It is going to append a "Cache-Control: nocache" as well, since this type of traffic is supposed to be personnal and we don’t want any shared cache on the internet to cache it.
- the statement `cookie xx` on the server line definition provides the value of the cookie inserted by HAProxy. When the client comes back, then HAProxy knows directly which server to choose for this client.

For more info refer:

https://www.haproxy.com/blog/load-balancing-affinity-persistence-sticky-sessions-what-you-need-to-know/

http://www.haproxy.org/#docs

   [Docker]: <https://www.docker.com/>

