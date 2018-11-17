# Enabling haproxy stats

In this example we gonna enable Haproxy stats.

#### Pre-requisite
- A Virtual Machine(VM) running any OS centos, ubuntu, etc.
- [Docker] installed on the VM.
- Port 7979 opened in the security group.

#### Steps to run HAProxy with stats enabled
- First we create docker image for HAProxy.
```sh
docker build -t haproxy:stats -f Dockerfile .
```
- Now we run HAProxy.
```sh
docker run -d -it -p 8080:80 -p 7979:7979 haproxy:stats
```
- Now we hit haproxy stats UI. URL `http://<machine-ip>:7979`. The browser should show the stats UI.
![alt text](https://raw.githubusercontent.com/milindchawre/haproxy_by_examples/master/stats/img/haproxy_stats.png) 

#### How it works
- The Haproxy stats are enabled using below configuration in haproxy.cfg file
```sh
listen stats                         # Define a listen section called "stats"
    bind *:7979                      # Listen on localhost:7979
    mode http
    stats enable                     # Enable stats page
    stats hide-version               # Hide HAProxy version
    stats refresh 10s                # Refresh stats every 10s
    stats realm Haproxy\ Statistics  # Title text for popup window
    stats uri /                      # Stats URI
    stats auth admin:admin           # Authentication credentials
```
- The comments in the config are self-explanatory.

For more info refer:

https://www.datadoghq.com/blog/how-to-collect-haproxy-metrics/

https://tecadmin.net/how-to-configure-haproxy-statics/

http://www.haproxy.org/#docs

