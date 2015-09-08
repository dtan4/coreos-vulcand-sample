# coreos-vulcand-sample

Zero-downtime Blue-Green deployment on CoreOS + [Vulcand](https://github.com/mailgun/vulcand)

Vulcand version: `v0.8.0-beta.2`

## HOW TO TEST

At this example, we try to deploy a very simple web application `coreos/example` to CoreOS cluster.

### Set up

Add `172.17.8.101 example.com` to `/etc/hosts` on your host machine,

```
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1       localhost
255.255.255.255 broadcasthost
::1             localhost

172.17.8.101 example.com
```

Launch CoreOS VMs and log in,

```bash
$ vagrant up
$ vagrant ssh core-01
```

### Deploy v1 containers

Run web application __v1__ containers,

```bash
$ docker run -d --name example-v1.1 -p 8086:80 coreos/example:1.0.0
$ docker run -d --name example-v1.2 -p 8087:80 coreos/example:1.0.0
```

Configure Vulcand to proxy to __v1__ container,

```bash
# Register v1 containers
$ etcdctl set /vulcand/backends/v1/backend '{"Type": "http"}'
$ etcdctl set /vulcand/backends/v1/servers/v1.1 '{"URL": "http://172.17.8.101:8086"}'
$ etcdctl set /vulcand/backends/v1/servers/v1.2 '{"URL": "http://172.17.8.101:8087"}'

# To proxy to v1 containers
$ etcdctl set /vulcand/frontends/example/frontend '{"Type": "http", "BackendId": "v1", "Route": "Host(`example.com`) && Path(`/`)"}'
```

Then access to `example.com` and you can see the current version _1.0.0_ .

![example_com](https://cloud.githubusercontent.com/assets/680124/9721329/a21893b0-55d3-11e5-88de-1b0c45394076.png)

### Deploy v2 containers

OK, let's deploy the new version __v2__.

Run web application __v2__ containers,

```bash
$ docker run -d --name example-v2.1 -p 8088:80 coreos/example:2.0.0
$ docker run -d --name example-v2.2 -p 8089:80 coreos/example:2.0.0
```

Configure Vulcand to proxy to __v2__ container,

```bash
# Register v2 containers
$ etcdctl set /vulcand/backends/v2/backend '{"Type": "http"}'
$ etcdctl set /vulcand/backends/v2/servers/v2.1 '{"URL": "http://172.17.8.101:8088"}'
$ etcdctl set /vulcand/backends/v2/servers/v2.2 '{"URL": "http://172.17.8.101:8089"}'

# To proxy to v2 containers
$ etcdctl set /vulcand/frontends/example/frontend '{"Type": "http", "BackendId": "v2", "Route": "Host(`example.com`) && Path(`/`)"}'
```

Then access to `example.com` and you can see the current version _2.0.0_ .

![example_com](https://cloud.githubusercontent.com/assets/680124/9721333/aeb836e8-55d3-11e5-9ecf-1eb707fcd81b.png)
