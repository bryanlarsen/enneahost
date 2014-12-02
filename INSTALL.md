# Installation

The current version of Enneahost uses Upstart and has only been tested on Ubuntu 14.04.

## Using Vagrant

Make sure you have [vagrant](http://vagrantup.com) and [ansible](http://ansible.com) installed on your development machine.   Unfortunately, ansible is not available for Windows.  If there is demand, I can modify the Vagrantfile so that ansible is run from within the vagrant box rather than outside.

    git submodule init
    git submodule update
    vagrant up

After your vagrant image has been provisioned, follow the instructions printed by vagrant to install your SSH keys.

## Using Ansible

TBD.

## Manually

### Upstart

Currently EnneaHost relies on Upstart, so should be installed on an upstart based Linux distribution.   Ubuntu 14.04 is the only one that has been tested.

As far as I can tell, systemd could work better than upstart.   Patches to use systemd instead of upstart would be gleefully accepted.

### Docker

Install [Docker](https://docker.com).

Ensure that `registry.service.consul:5000` is added to the Docker insecure registry whitelist.   Add `--insecure-registry=registry.service.consul:5000` to DOCKER_OPTS in /etc/default/Docker.

Ensure that /var/lib/docker/vfs/dir and all of its parents are readable by the nginx www-data user.

### Consul

Install [Consul](https://consul.io).  You may either install it directly on the host or inside of a docker container.  If you do to install it inside of a container, ensure that you set the hostname of the container the same as the hostname of the host so that registrator behaves as designed.

Ensure that the host and docker containers can access consul via standard DNS name resolution.  In other words, `ping consul.service.consul.` should work. See https://www.morethanseven.net/2014/04/25/consul/ for Ubuntu 14.04 instructions.

Ensure that the advertised address is the externally visible address.

    docker run --hostname="$(hostname)" -v /var/consul-data:/data -p 8400:8400 -p 8500:8500 -p 8600:53/udp -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8301:8301/udp -p 8302:8302/udp progrium/consul:latest -server -bootstrap-expect=1 -client=0.0.0.0 -advertise=172.17.42.1

### consul-template

Install the [consul-template](https://github.com/hashicorp/consul-template) executable somewhere in root's PATH.

### ennea cli (optional)

Install the [ennea](https://raw.github.com/bryanlarsen/enneahost/master/bin/ennea) shell script into your $PATH and make it executable.

### ennea-git-builder (optional)

[ennea-git-builder](https://github.com/bryanlarsen/ennea-git-builder)

### ennea-webapp-nginx (optional)

Install nginx and then [ennea-webapp-nginx](https://github.com/bryanlarsen/ennea-webapp-nginx)

## Manual Installation Part 2: Use EnneaHost to bootstrap the rest.

### Docker Registry

Set the consul key `enneahost/images/registry` to

    {
      "namespace": "library",
      "run_options": "--volume=/var/docker-registry:/tmp/registry -p 5000:5000"
    }

If you installed Enneahost using Vagrant, then the Consul UI is available at `http://192.168.33.100:8500/`.

Once you set the key, the registry container will be pulled and run.  This can take a while.  You can tail the log in `/var/log/upstart/docker-registry.log` if you're curious.

### progrium/registrator (optional)

[see https://github.com/progrium/registrator](https://github.com/progrium/registrator)

Registrator automatically registers running Docker containers with the Consul service catalog.

Set the consul key `enneahost/images/regsistrator` to

    {
      "namespace": "progrium",
      "run_options": "--volume=/var/run/docker.sock:/tmp/docker.sock",
      "run_cmd": "consul://consul.service.consul:8500"
    }

### bryanlarsen/docker-plugin-kv-consul (optional)

[see https://github.com/bryanlarsen/docker-plugin-kv-consul](https://github.com/bryanlarsen/docker-plugin-kv-consul)

Allows you to automatically set consul keys by adding ENV statements to your Dockerfile or Docker run options

Set the consul key `enneahost/images/kv-consul` to

    {
      "namespace": "bryanlarsen",
      "run_options": "--volume=/var/run/docker.sock:/var/run/docker.sock -e KV_CONSUL_IP=consul.service.consul"
    }

