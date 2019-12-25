# Kubernetes testing configuration

This repository contains configuration which I'm using to create a bare metal
kubernetes cluster. With this configuration I'm getting to learn the ins and
outs of kubernetes.

Kubernetes so far is a complex beast compared to a simple standalone
docker-compose configuration. There are lots of new concepts and documenting
those are outside the scope of this project.

This project isjust for fun and learning. If you're looking for a tutorial,
look elsewhere.

## Cluster configuration

### Vagrant

When vagrant is installed, the cluster can be easily configured with the

```bash
vagrant up
```

in the root of this project. It'll spin up an Ubuntu 18.04 virtualbox machine
and will automatically install kubernetes. Also, a network policy provider (not
sure if it's called like this, but I believe so) called
[calico](https://docs.projectcalico.org/v3.11/getting-started/kubernetes/) is
automatically installed. This is a vital piece of the kubernetes cluster.

### Network setup

In the Vagrantfile a host local network is configured. The master node will get
IP address 192.168.34.10.

## Test configurations

### Traefik

Traefik is my goto proxy for my development setups. So far I've only used it
with docker as a configuration provider. As it also supports kubernetes as an
ingress controller and loadbalancer, I figured I should figure this out as
well. In the [traefik subdirectory](./resources/traefik) the deployment
configuration can be found.

The configuration is based on the
[traefik kubernetes guide with letsencrypt](https://docs.traefik.io/v2.1/providers/kubernetes-crd/),
but without the letsencrypt part (which I currently find irrelevant).

### Cron job

Kubernetes supports cron jobs to be run in a cluster. A sample configuration
can be found [here](./resources/cronjob.yml)
