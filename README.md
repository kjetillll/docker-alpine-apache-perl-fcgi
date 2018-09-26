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

	#rm -rf /web   #to start anew
	mkdir /web
	docker run -d -p 80:80 -p 443:443 -v /web:/web --name fcgi \
	       kjetils/alpine-apache-perl-fcgi

This will publish the containers ports 80 (http) and 443 (https) to
the host so that when port 80 (or 443) is contacted from the world
outside, it is redirected to apache2 inside the container.

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

To use this start the container with:

   mkdir -p /path/to/web
   docker run -d --name fcgi -p 80:80 -p 443:443 -v /path/to/web:/web kjetils/alpine-apache-perl-fcgi

#### Successsful setup

Checks for successful setup of the CGI and FCGI test.pl scripts:

       curl 127.0.0.1/hello
       time for i in {1..10}; do curl 127.0.0.1/cgi-bin/test.pl; done  #CGI
       time for i in {1..10}; do curl 127.0.0.1/fcgi-bin/test.pl; done #FCGI
       for i in {1..10}; do curl 127.0.0.1/hello; done #FCGI alias to test.pl
       for i in {1..10}; do curl 127.0.0.1/bye; done   #FCGI another alias
