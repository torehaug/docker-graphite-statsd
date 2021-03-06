[![Docker Hub](http://img.shields.io/badge/docker-hub-brightgreen.svg?style=flat)](https://registry.hub.docker.com/u/hopsoft/graphite-statsd/)
[![Gratipay](http://img.shields.io/badge/gratipay-contribute-009bef.svg?style=flat)](https://gratipay.com/hopsoft/)

# Docker Image for Graphite, Statsd & Grafana

## Get Graphite & Statsd (and Grafana) running instantly

Graphite & Statsd can be complex to setup.
This image will have you running & collecting stats in just a few minutes.

This fork adds instructions to add [Grafana](http://grafana.org/) as an alternative web frontend as well - [have a look at some grafana examples](http://play.grafana.org/). Just the README has been changed - you need to run some additional commands in the docker image to get Grafana up and running.

## Quick Start

```sh
sudo docker run -d \
  --name graphite \
  --restart=always \
  -p 80:80 \
  -p 2003:2003 \
  -p 3000:3000 \
  -p 8125:8125/udp \
  -p 8126:8126 \
  hopsoft/graphite-statsd
```

This starts a Docker container named: **graphite**

That's it, you're done ... almost.

If you want to install [Grafana](http://grafana.org/) as an alternative web frontend run the following in the docker container ([from Installing on Debian / Ubuntu](http://docs.grafana.org/installation/debian/)):

Note: Check if a [newer version](http://grafana.org/download/) of Grafana has been released before running the commands below
```
$ wget https://grafanarel.s3.amazonaws.com/builds/grafana_2.1.3_amd64.deb
$ sudo apt-get install -y adduser libfontconfig
$ sudo dpkg -i grafana_2.1.3_amd64.deb

### In order to start grafana-server, execute
$ sudo update-rc.d grafana-server defaults 95 10
### In order to start grafana-server, execute
$ sudo service grafana-server start
```

### Includes the following components

* [Nginx](http://nginx.org/) - reverse proxies the graphite dashboard
* [Graphite](http://graphite.readthedocs.org/en/latest/) - front-end dashboard
* [Grafana](http://grafana.org/)* - alternative front-end dashboard for Graphite - not included in the image - must be installed (instructions above)
* [Carbon](http://graphite.readthedocs.org/en/latest/carbon-daemons.html) - back-end
* [Statsd](https://github.com/etsy/statsd/wiki) - UDP based back-end proxy

### Mapped Ports

| Host | Container | Service  |
| ---- | --------- | -------- |
|   80 |        80 | nginx    |
| 2003 |      2003 | carbon   |
| 3000 |      3000 | grafana* |
| 8125 |      8125 | statsd   |
| 8126 |      8126 | admin    |

### Mounted Volumes

| Host              | Container                  | Notes                           |
| ----------------- | -------------------------- | ------------------------------- |
| DOCKER ASSIGNED   | /opt/graphite              | graphite config & stats storage |
| DOCKER ASSIGNED   | /etc/nginx                 | nginx config                    |
| DOCKER ASSIGNED   | /opt/statsd                | statsd config                   |
| DOCKER ASSIGNED   | /etc/logrotate.d           | logrotate config                |
| DOCKER ASSIGNED   | /var/log                   | log files                       |

### Base Image

Built using [Phusion's base image](https://github.com/phusion/baseimage-docker).

* All Graphite related processes are run as daemons & monitored with [runit](http://smarden.org/runit/).
* Includes additional services such as logrotate.

## Start Using Graphite & Statsd

### Send Some Stats

Let's fake some stats with a random counter to prove things are working.

```sh
while true
do
  echo -n "example.statsd.counter.changed:$(((RANDOM % 10) + 1))|c" | nc -w 1 -u localhost 8125
done
<CTL-C>
```

### Visualize the Data

Open Graphite in a browser at [http://localhost/dashboard](http://localhost/dashboard).

### Visualize the Data - With Grafana

Open Grafana in a browser at [http://localhost:4000/](http://localhost:4000/) and add a Graphite datasource.

## Secure the Django Admin

Update the default Django admin user account. _The default is insecure._

  * username: root
  * password: root
  * email: root.graphite@mailinator.com

First login at: [http://localhost/account/login](http://localhost/account/login)
Then update the root user's profile at: [http://localhost/admin/auth/user/1/](http://localhost/admin/auth/user/1/)

## Secure the Grafana Admin

Update the default Grafana admin user account. _The default is insecure._

  * username: admin
  * password: admin

First login at: [http://localhost:4000](http://localhost:4000) then update the admin user's profile


## Change the Configuration

Read up on Graphite's [post-install tasks](https://graphite.readthedocs.org/en/latest/install.html#post-install-tasks).
Focus on the [storage-schemas.conf](https://graphite.readthedocs.org/en/latest/config-carbon.html#storage-schemas-conf)

1. Stop the container `docker stop graphite`.
1. Find the configuration files on the host by inspecting the container `docker inspect graphite`.
1. Update the desired config files.
1. Restart the container `docker start graphite`.

**Note**: If you change settings in `/opt/graphite/conf/storage-schemas.conf`
be sure to delete the old whisper files under `/opt/graphite/storage/whisper/`.

---

**Important:** Ensure your Statsd flush interval is at least as long as the highest-resolution retention.
For example, if `/opt/statsd/config.js` looks like this.

```
flushInterval: 10000
```

Ensure that `storage-schemas.conf` retentions are no finer grained than 10 seconds.

```
[all]
pattern = .*
retentions = 5s:12h # WRONG
retentions = 10s:12h # OK
retentions = 60s:12h # OK
```

## Statsd Admin Management Interface

A management interface (default on port 8126) allows you to manage statsd and retrieve stats 

```sh
# show all current counters
echo counters | nc localhost 8126
```

more info and additional commands: [admin_interface](https://github.com/etsy/statsd/blob/master/docs/admin_interface.md) 

## A Note on Disk Space

If running this image on cloud infrastructure such as AWS,
you should consider mounting `/opt/graphite` & `/var/log` on a larger volume.

1. Configure the host to mount a large EBS volume.
1. Specify the volume mounts when starting the container.

    ```
    sudo docker run -d \
      --name graphite \
      --restart=always \
      -v /path/to/ebs/graphite:/opt/graphite \
      -v /path/to/ebs/log:/var/log \
      -p 80:80 \
      -p 2003:2003 \
      -p 8125:8125/udp \
      hopsoft/graphite-statsd
    ```

## Additional Reading

* [Introduction to Docker](http://docs.docker.io/#introduction)
* [Official Statsd Documentation](https://github.com/etsy/statsd/)
* [Practical Guide to StatsD/Graphite Monitoring](http://matt.aimonetti.net/posts/2013/06/26/practical-guide-to-graphite-monitoring/)
* [Configuring Graphite for StatsD](https://github.com/etsy/statsd/blob/master/docs/graphite.md)

## Contributors

Build the image yourself.

### OSX

1. `git clone https://github.com/hopsoft/docker-graphite-statsd.git`
1. `cd docker-graphite-statsd`
1. `vagrant up`
1. `vagrant ssh`
1. `sudo docker build -t hopsoft/graphite-statsd /vagrant`

**Note**: Pay attention to the forwarded ports in the [Vagrantfile](https://github.com/hopsoft/docker-graphite-statsd/blob/master/Vagrantfile).

### Linux

1. `git clone https://github.com/hopsoft/docker-graphite-statsd.git`
1. `sudo docker build -t hopsoft/graphite-statsd ./docker-graphite-statsd`
