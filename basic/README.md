# Simple Load Balancing using haproxy

In this example we gonna load balance [2 apache webservers] using [HAProxy]. The overall architecture would look like this below.
![alt text](https://raw.githubusercontent.com/milindchawre/haproxy_by_examples/master/basic/img/haproxy_basic.png)

#### Pre-requisite
- A Virtual Machine(VM) running any OS centos, ubuntu, etc.
- [Docker] installed on the VM.
- Ports 8080-8082 opened in the security group.

#### Steps to create simple load balancing using HAProxy
- First we create docker image for our 2 apache webserver using these commands.
```sh
docker build -t apache1 -f Dockerfile1 .
docker build -t apache2 -f Dockerfile2 .
```
- Now we create docker image for HAProxy.
```sh
docker build -t haproxy -f Dockerfile3 .
```
- Now we run our apache webserver.
```sh
docker run -d -it -p 8081:80 apache1
docker run -d -it -p 8082:80 apache2
```
- Verify the webserver is running by hitting this URL in browser. URLs `http://<machine-ip>:8081` and `http://<machine-ip>:8082` The browser should show `server-1` or `server-2` as an output.
- Now run HAProxy.
```sh
docker run -d -it -e machine_ip=<ip-of-machine> -p 8080:80 haproxy
```
- Now hit port 8080 where HAProxy is load balancing the webservers. URL `http://<machine-ip>:8080`. The browser should now output `server-1` and `server-2` one after other in roundrobin fashion. So HAProxy is hitting the one of the 2 webservers in each turn.

#### How it works
- This simple Load balacing is achieved by letting HAProxy know how to load balance through its configuration. In haproxy.cfg file we have defined it
```sh
frontend apache_frontend
  bind *:80
  use_backend apache_backend

backend apache_backend
   balance roundrobin
   server web1 ${machine_ip}:8081 check
   server web2 ${machine_ip}:8082 check
```
- A **backend** `backend apache_backend` is a set of servers that receives forwarded requests. The Backend consist of: 
    - which load balance algorithm to use (eg. **roundrobin**)
    - a list of servers and ports (the machine-ip and port on which webservers are running)
- A **frontend** `frontend apache_frontend` defines how requests should be forwarded to backends. It consist of:
    - a set of IP addresses and a port (e.g. 10.0.10.10:80, *:443, etc.)
    - use_backend rules, which define which backends to use.


For more info refer:

https://www.digitalocean.com/community/tutorials/an-introduction-to-haproxy-and-load-balancing-concepts

http://www.haproxy.org/#docs

   [Docker]: <https://www.docker.com/>
   [2 apache webservers]: <https://httpd.apache.org/>
   [HAProxy]: <http://www.haproxy.org/>

