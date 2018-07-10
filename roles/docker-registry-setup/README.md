# Docker Registry Setup for Overcloud Container Images

## Usage
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html-single/getting_started_with_containers/index

To use this role, call it with something such as:

---
- hosts: registry
  tasks:
    - import_role:
        name: docker-registry-setup
...

## Workflow
The host will be configured as a docker registry

There are some built-in checks to see if this target host is already configured as a docker registry. If so, it will skip the configuration and display a message that a variable called 'docker_registry_setup_force' can be set to True to allow the docker registry to be updated.

There is also a check duing the docker configuration if /etc/syscoinfig/docker has already been modified to point to itself and the upstream repo, it will not re-configure this host.

## Verifying
Pull an image:
[root@undercloud stack]# docker pull <hostname>:5000/rhosp13/openstack-iscsid:latest
Trying to pull repository 172.16.0.1:8787/rhosp13/openstack-iscsid ...
13.0-38: Pulling from 172.16.0.1:8787/rhosp13/openstack-iscsid
e0f71f706c2a: Pull complete
121ab4741000: Pull complete
2824d3a8e244: Pull complete
edb8df6b610c: Pull complete
Digest: sha256:69e2bd1af17c609a42a2aadd2632f87b94d697e6f8036d747486f1f59fcf062e
Status: Downloaded newer image for 172.16.0.1:8787/rhosp13/openstack-iscsid:13.0-38

[root@undercloud stack]# docker images
REPOSITORY                                 TAG                 IMAGE ID            CREATED             SIZE
172.16.0.1:8787/rhosp13/openstack-iscsid   13.0-38             47f41decc493        3 weeks ago         458 MB

## Syncing
Normally the local registry will sync with upstream. To disable this set 'docker_registry_sync: False'

This also happens to be a good way to do a read-only check on an undercloud to see if it has a running registry and list the images with -v

