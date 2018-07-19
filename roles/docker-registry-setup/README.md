# Docker Registry Setup for OpenStack Overcloud Container Images
This role is intended to be run on CentOS/RHEL but may work with Fedora or other Linux distributions. It sets up a docker registry on the target host, then contacts an upstream (can be CDN or another internal registry) and syncs a list of images from it, then can serve those out. The default variables in the role have the required images for an OpenStack OverCloud install, but could be any images. Steps taken from https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html-single/getting_started_with_containers/index

## Define your environment
The 'roles/docker-registry-setup/defaults/main.yml. can be set to whatever is appropriate in your environment. Here are a two sample scenarios. 

The first scenario uses the Red Hat Registry CDN as the 'upstream' registy and a local RHEL server as 'local' and configures a registry on it. Now this Local Registry is able to serve images out.

```
   upstream                        local
+-------------+                +-------------+
|             |                |             |
| Red Hat     |                |  Local      |
| Registry    +--------------->+  Registry   |
| (CDN)       |                |             |
|             |                |             |
+-------------+                +-------------+
```

This next scenario actually uses the previously configured Local Registry as the new 'upstream' for this scenario. The 'local' registry in this case is an Undercloud server. 

```
   upstream                        local
+-------------+                +-------------+
|             |                |             |
| Local       |                |  Undercloud |
| Registry    +--------------->+  Registry   |
|             |                |             |
|             |                |             |
+-------------+                +-------------+
```

Note that though the same role is used in both cases, there are actually different tasks used for setup of a standard docker registry and an Undercloud registry, but the end result is the same. One example is that the 'openstack' CLI is used to geenrate a list of required images, which is present on an Undercloud only.

## Usage
To use this role, call it with something such as:

---
- hosts: registry
  tasks:
    - import_role:
        name: docker-registry-setup
...

## Workflow
The host will be configured as a docker registry

There are some built-in checks to see if this target host is already configured as a docker registry. If so, it will skip the configuration and display a message that a variable called 'docker_registry_setup_force' can be set to 'true' to allow the docker registry to be updated.

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
