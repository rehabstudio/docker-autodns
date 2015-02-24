![License MIT](https://img.shields.io/badge/license-MIT-blue.svg)

docker-autodns sets up a container running dnsmasq and docker-gen. The
container uses docker-gen to reconfigure dnsmasq as other containers are
started and stopped. Configured correctly, this container will allow other
containers to address each other by name alone, e.g. If you're running a
container with the name `mycontainer` which is serving a HTTP API on port 8080,
then from another container on the same host you can simply
`curl http://mycontainer:8080` to contact it.

autodns is a simple solution for a specific usecase and does not attempt to
replace more fully featured systems like
[skydns](https://github.com/skynetservices/skydns) which you should definitely
check out if autodns doesn't meet your needs.


### Prerequisites

In order for autodns to work, the docker daemon must be started with a few
special options, you'll need to explicitly specify a bridge IP and tell docker
to use that for DNS resolution inside containers. It's also useful to
explicitly specify some external DNS servers should your local DNS service be
unreachable for any reason.

```bash
$ docker -d --bip=172.17.42.1/16 --dns=172.17.42.1 --dns=8.8.8.8 --dns=8.8.4.4
```

If you're using an Upstart based system (Ubuntu/Debian, at least until the
systemd switchover), then the easiest way to do this is to modify docker's
startup scripts inside `/etc/init/`. You should ensure that
`/etc/default/docker` contains the following line (note the inclusion of
Google's public DNS servers):

```bash
DOCKER_OPTS="--bip=172.17.42.1/16 --dns=172.17.42.1 --dns=8.8.8.8 --dns=8.8.4.4"
```

and then just restart the docker service:

```bash
$ sudo service docker restart
```

Instructions for other init systems (sysV/systemd/etc) may differ. These will
hopefully be added soon.

### Usage

To run it:

```bash
$ docker run -d -p 0.0.0.0:53:53/udp -v /var/run/docker.sock:/var/run/docker.sock rehabstudio/autodns
```

Then start any containers you need, no special options are required:

```bash
$ docker run -d --name mycontainer somerepo/somecontainer
```

You should then be able to use the local DNS service from inside a container:

```bash
$ docker run -ti ubuntu bash
$ ping mycontainer
64 bytes from mycontainer (172.17.0.25): icmp_seq=1 ttl=64 time=0.075 ms
64 bytes from mycontainer (172.17.0.25): icmp_seq=1 ttl=64 time=0.042 ms
```


### Building autodns locally

If you're planning to customise autodns, whether to submit a patch or just to
customise a private build, the easiest way to do so is to clone this repo
locally, build a private copy and push to a docker registry using a different
repo/name.

Building this container uses docker's standard process, nothing special:

```bash
$ docker build -t autodns .
```

As always, pull requests are very welcome.
