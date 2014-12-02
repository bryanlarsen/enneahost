# EnneaHost - a simple Docker PaaS using Consul

## Simple

The core of EnneaHost is only around 100 lines.  It and the optional
components are very trivial.  Easy to understand, and easy to adapt to
your needs.  None of them do very much, but together the whole is
greater than the sum of the parts.

## Independent Components

EnneaHost is built out of several third-party components and several
first-party ones.   All are independently useful outside of EnneaHost.

## Stable

Uses stable Docker containers as its base rather than over-relying on
deployment systems such as Ansible or Chef, which tend to bit-rot
quite quickly.  It provides a Heroku style deployment system (git push
your repository to build it), but that's optional and secondary.
Build your containers today and they will still work 5 years from now.

## Forward Compatible

Right now the PaaS space is very dynamic: Deis, Flynn, Octohost and
Dokku are just some of the open source contenders, and there are
several other closed source systems.  Yet none of these are built on
new exciting technologies such as Kubernetes and Mesos.  None of them
are very mature.

EnneaHost assumes that it isn't going to "win".  Instead it seeks to
make your transition costs very minimal.  We just assume that both
Docker and Consul are going to "win", and use those two components to
do most of the heavy lifting.  We also attempt to use these two
components in a fashion that makes moving away from EnneaHost really
simple.

We've tried to keep the EnneaHost metadata in a form that makes it
useful for documentation on how to run the components without
EnneaHost.  For example, if you distribute a Dockerfile, the
`DOCKER_RUN_OPTIONS` line should be immediately useful to somebody who
is running the container manually, even if they don't know what
EnneaHost is.

## Single Host Compatible

EnneaHost will deploy and work on a single host.  However if you have
multiple hosts, EnneaHost will provide redundancy and load balancing.

## Tons of Languages and Frameworks Supported

[tutum buildstep](https://github.com/tutumcloud/buildstep) uses Heroku
build packs to support tons of different languages and frameworks.

Not only web applications are supported.  Virtually anything that can
be put into a Docker container can be run by EnneaHost.

## The Name

Ennea is the ancient Greek prefix for ninth, in the same way that octo is the
prefix for eighth.  EnneaHost started as a fork of OctoHost, and owes
it a great debt.  I do hope that OctoHost liberally steals back from
EnneaHost.

# How to use

There are two ways to start a container on Enneahost: directly or via git push.

## Directly

Store an empty JSON object "{}" in the consul key
"enneahost/images/<name>".  Enneahost will then automatically run the
named Docker image.

Optional attributes that may be set in the JSON object:

- namespace: default "registry.service.consul:5000". The Docker namespace.  This is either the location of a private Docker repository, or a prefix on the Docker Hub.   Use "library" to pull official Docker images.

- tag: default "latest".

- run_options: default "".   These are the options that are passed to Docker.  Do not use "-d" or similar options.  These options are sent through [consul-template](https://github.com/hashicorp/consul-template) to allow dynamic customization.   Consul-template will automatically restart your container if any dynamic configuration changes.

- run_cmd: default "".  The options that are passed to the executable inside the Docker container.   These also support consul-template customization.

## Using the Optional Git Builder

Make sure your repository contains a Dockerfile.   Push your repository to the master branch on your EnneaHost.   The repository will be automatically built, pushed to a repository and started.

    git remote add enneahost git@<ip>:<name>
    git push enneahost <branch>:master

You may set the environment variables "DOCKER_RUN_OPTIONS" and "DOCKER_RUN_CMD" in your Dockerfile to customize.   See above for details.

# Installation

To install Enneahost we first must install its core dependencies (Docker, Consul and consul-template), and then we'll install Enneahost and then we'll use Enneahost to install the remaining  dependencies.

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

# The EnneaHost Components

## [ennea-upstart](https://github.com/bryanlarsen/ennea-upstart)

This is the core of EnneaHost.  It starts docker containers when a key is added to `enneahost/images/` in consul.

## [webapp-nginx](https://github.com/bryanlarsen/ennea-webapp-nginx)

Creates a load-balancing nginx proxy configuration for a webapp

## ennea cli

- `ennea <name> run <cmd...>`: run cmd in new container (docker run)
- `ennea <name> exec <cmd...>`: run cmd in existing container (docker exec)

## [git builder](https://github.com/bryanlarsen/ennea-git-builder)

Builds and pushes docker images on a git push

# Example: Hosting a Rails Application

## Postgres

Set the consul key `enneahost/images/postgres` to

    {
      "namespace": "library",
      "run_options": "-p 5432:5432"
    }

For production use we highly recommend the use of `--volume=<path>:/var/lib/postgresql/data` to make it harder to accidentally trash all your databases.

## Rails

For the sake of simplicity we're going to use a Heroku buildpack to Dockerize our Rails application.   This is an easy, but heavy, solution.

Dockerfile:

   FROM tutum/buildstep

   RUN /start assets

   EXPOSE 80
   ENV PORT 80
   CMD ["/start", "web"]
   VOLUME /app/public

   ENV KV_SET:webapp/#<IMAGE_NAME># foo
   ENV KV_SET:#<IMAGE_NAME>#/cname example.com
   ENV KV_SET:#<IMAGE_NAME>#/public #jq<.Volumes["/app/public"]>#

   ENV DOCKER_RUN_OPTIONS -P --env=DATABASE_URL=postgres://postgres@postgres.service.consul/#<IMAGE_NAME>#-production

The funny `#<IMAGE_NAME>#` syntax takes advantage of kv-consul to dynamically insert the actual image name.   This is usually extraneous: I would recommend just replacing `#<IMAGE_NAME>#` with your application name to keep things clear.   Just make sure you use the same name when pushing to enneahost.

The KV_SET:webapp line triggers the [ennea webapp plugin](https://github.com/bryanlarsen/ennea-webapp-nginx).  The cname and public lines are parameters for the plugin.

Then push your app to enneahost:

    git remote add enneahost git@<ip>:<name>
    git push enneahost master

Give it a few minutes to build the image and push to the repository.   After it has been pushed to the repository your `git push` command will return.   Then give it a few more minutes for EnneaHost to download the image from the repository and for your Rails app to start running.

You then should be able to initialize your database:

    ennea <name> run rake db:create
    ennea <name> run rake db:migrate
    ennea <name> run rake db:seed

Your application should now be running on your host and should be available at the cname you provided as well as <name>.<ip>.xip.io.
