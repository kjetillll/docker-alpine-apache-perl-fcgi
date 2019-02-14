## Alpine microcontainer with Apache2, perl5 and FCGI.pm

This is a micro docker container ![](https://images.microbadger.com/badges/image/nimmis/alpine-apache.svg) based on Alpine OS and Apache version 2.

The image:

...is built on [nimmis/alpine-micro](https://hub.docker.com/r/nimmis/alpine-micro/)
![](https://images.microbadger.com/badges/image/nimmis/alpine-micro.svg)

...but is heavily based on (basically a modified copy of) [nimmis/alpine-apache](https://hub.docker.com/r/nimmis/alpine-apache/)
![](https://images.microbadger.com/badges/image/nimmis/alpine-apache.svg)

Visit
<https://hub.docker.com/r/nimmis/alpine-micro/>,
<https://hub.docker.com/r/nimmis/alpine-apache/> or
<https://hub.docker.com/r/kjetils/alpine-apache-perl-fcgi/>.

#### starting the container as a daemon

Example, starting the container:

    #rm -rf /web; mkdir /web     #to start anew
    docker run -d -p 80:80 -p 443:443 -v /web:/web --name fcgi kjetils/alpine-apache-perl-fcgi

This will publish the containers ports 80 (http) and 443 (https) to
the host so that when port 80 (or 443) is contacted from the world
outside, it is redirected to apache2 inside the container.

The /web folder can be empty at start. The first time you run the
container it will be populated with files and folders from the
container.

These files inside /web can be changed (the container will not
overwrite them) and new files and folders can be added for your own app.

	docker exec -ti fcgi ps        #view processes, httpd and others
	docker exec -ti fcgi /bin/sh   #run commands

#### Static web folder

The images exposes a volume at /web. The structure is

| Directory | Function |
| --------- | -------- |
| /web/html | web root |
| /web/cgi-bin | cgi-bin folder |
| /web/fcgi-bin | fcgi-bin folder |
| /web/config | apache config directory |
| /web/logs | apache log directory with access.log and error.log |
| /web/internal | internal pages, error pages etc

Start the container:

    mkdir -p /path/to/web

    docker run -d --name fcgi -p 80:80 -p 443:443 -v /path/to/web:/web kjetils/alpine-apache-perl-fcgi

#### Successsful setup

Checks for successful setup of the CGI and FCGI test.pl scripts:

    cat /web/fcgi-bin/test.pl                   #view hello app / test app
    cat /web/config/conf.d/fcgid.conf           #view routes

    curl 127.0.0.1/hello                        # 1
    curl 127.0.0.1/hello                        # 2
    curl 127.0.0.1/hello                        # 3
    time for i in {1..10}; do curl -s 127.0.0.1/cgi-bin/test.pl; done  #CGI  1 1 1 1 1 1 1 1 1 1
    time for i in {1..10}; do curl -s 127.0.0.1/fcgi-bin/test.pl; done #FCGI 1 2 3 4 5 6 7 7 8 10

Increase 10 to 1000 and you'll see that FCGI is faster than CGI.
Especially when you have initialization (like connecting to a
database) which will not be repeated much for FCGI as with CGI.
