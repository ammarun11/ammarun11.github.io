---
title: "[Openshift]Setup and Installation Openshift RedHat Codeready"
date: 2021-09-06
categories: [ngoprek, server, cloud, openshift, container]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Setup and Installation Openshift RedHat Codeready

### Minimum requirements
CodeReady Containers requires the following system resources:

- 4 physical CPU cores
- 9 GB of free memory
- 35 GB of storage space

Detail Environtment we are using:

* am-codeready (10.20.10.65) : 8 vCPU, 12 GB RAM, 80 GB HDD1 OS 
* CentOS 8 Stream
* OpenShift version: 4.8.4

### Required software packages for Linux

```s
Red Hat Enterprise Linux/CentOS

su -c 'yum install NetworkManager'
```

### Installing Openshift RedHat CodeReady

0. Make sure you're setup with non-root user, then give non-root user access to manage systemd
```
# su student

$ export XDG_RUNTIME_DIR=/run/user/`id -u`
$ sudo systemctl restart systemd-logind.service
$ loginctl enable-linger student
$ export XDG_RUNTIME_DIR=/run/user/$(id -u)
```

1. download crc binary https://console.redhat.com/openshift/create/local

```
wget -c https://developers.redhat.com/content-gateway/rest/mirror/pub/openshift-v4/clients/crc/latest/crc-linux-amd64.tar.xz
```

2. On Red Hat Enterprise Linux/CentOS 8, assuming the archive is in the ~/Downloads directory, follow these steps:

    Extract the contents of the archive:
```
$ cd ~/Downloads
$ tar xvf crc-linux-amd64.tar.xz
Create the ~/bin directory if it does not exist and copy the crc executable to it:

$ mkdir -p ~/bin
$ cp ~/Downloads/crc-linux-*-amd64/crc ~/bin
Add the ~/bin directory to your PATH:

$ export PATH=$PATH:$HOME/bin
$ echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
```

3. Setting up CodeReady Containers

    The crc setup command performs operations to set up the environment of your host machine for the CodeReady Containers virtual machine.

    This procedure will create the ~/.crc directory if it does not already exist.
Set up your host machine for CodeReady Containers:

```
Replace the earlier crc executable with the executable of the latest release. Verify that the new crc executable is in use by checking its version:

$ crc version

Set up the new CodeReady Containers release:

$ crc setup

Start the new CodeReady Containers virtual machine:

$ crc start
Started the OpenShift cluster.

The server is accessible via web console at:
  https://console-openshift-console.apps-crc.testing

Log in as administrator:
  Username: kubeadmin
  Password: 6Hd7K-u2VY8-2S559-rBMWt

Log in as user:
  Username: developer
  Password: developer

Use the 'oc' command line interface:
  $ eval $(crc oc-env)
  $ oc login -u developer https://api.crc.testing:6443

```

4. Accessing the OpenShift cluster

```
Run crc console. This will open your web browser and direct it to the web console.
Log in as the developer user with the password printed in the output of the crc start command.

$ eval $(crc oc-env)

$ crc console --credentials
To login as a regular user, run 'oc login -u developer -p developer https://api.crc.testing:6443'.
To login as an admin, run 'oc login -u kubeadmin -p 6Hd7K-u2VY8-2S559-rBMWt https://api.crc.testing:6443'
```

### Verify Access admin dashboard
![Dashboard2](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/1-Dashboard-Openshift-codeready.png)

![Dashboard2](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/2-Dashboard-Openshift-codeready.png)


**Referensi**
* [https://access.redhat.com/documentation/en-us/red_hat_codeready_containers/1.31/html/getting_started_guide/index](https://access.redhat.com/documentation/en-us/red_hat_codeready_containers/1.31/html/getting_started_guide/index)
* [https://crc.dev/crc/#introducing-codeready-containers_gsg](https://crc.dev/crc/#introducing-codeready-containers_gsgs)

# Happy,  Enjoy Ngoprek ~
