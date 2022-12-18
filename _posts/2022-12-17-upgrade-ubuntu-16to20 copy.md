---
title: "[Ubuntu] Upgrade Ubuntu 16 LTS to Ubuntu 20 LTS"
date: 2022-12-17
categories: [ngoprek, server, cloud, ubuntu, storage]
tags:
  - Jekyll
  - update
---
## Investigating

### we will follow this step 16 > 18 > 20 LTS

Before the 16.04 to 18.04 update
First off, caveat emptor. This is what I did, and it worked. If you do the same and it breaks something, don’t blame me. Also note that I am brazen and always run sudo -s before I get started. If you don’t like to do that, you may need to prefix some of these commands with sudo in order for them to work.

It’s probably also a good idea first to do a full backup of your server that you can restore from if things go totally off the rails.

Yes, this is going to be a two-step upgrade. First we’ll upgrade from Ubuntu 16.04 LTS to 18.04 LTS. Then we can do the 18.04 LTS to 20.04 LTS upgrade to get ourselves current. Then we won’t have to think about this for another 3 1/2 years! Whew!

There’s some stuff to do before you run that do-release-upgrade command. First, edit /etc/apt/sources.list in your text editor of choice (I like nano) and add these lines at the end:

```
deb http://archive.ubuntu.com/ubuntu/ xenial main universe multiverse
deb http://archive.ubuntu.com/ubuntu/ xenial-security main universe multiverse
```

Save that. Then if you’re running ufw, which you should be, do this to give yourself a back door in case something goes wrong (assuming you’re running this update over ssh, which of course Ubuntu will warn you is a Bad Idea).

```
ufw allow 1022
```

Hold on! Failure may be looming in the near future. Before doing anything else, try this:

```
apt-get update
apt-get upgrade
```

*OPTIONAL
I almost always run apt-get dist-upgrade instead of just apt-get upgrade because I always assumed the former does everything the latter does, and more. Not so. I’ve run into a problem on some servers with messages that cloud-init was held back. My fix for that is to do this:

```
apt-mark unhold cloud-init
apt-get upgrade
reboot
```

When things come back, let’s do this:
```
apt-get dist-update
apt-get autoremove
```

Then proceed as normal…

Now we’re ready to get started. Run these commands:
```
do-release-upgrade -c
do-release-upgrade
```

Don’t walk away at this point… you’re going to have to answer some prompts along the way. In general I would always recommend keeping your existing versions of any files it asks you about. Once all of that is over, your server will reboot, and you should be able to log back into a fully functioning Ubuntu 18.04 LTS install. I would recommend testing whatever services you have running on the server, just to be sure everything is working properly, before you continue to the 20.04 LTS upgrade.

After the 16.04 to 18.04 update, before the 18.04 to 20.04 update
Run all of the regular updates, just to be sure you’re fully current. If you’re brazen like me you’ll tackle that with one command:

```
apt update; apt -y dist-upgrade; apt -y autoremove; reboot
```

*OPTIONAL
Once you’re back up and running, throw this in. I don’t recall why I needed it now, and if you’re not using cURL you may not need it at all, but anyway, I did this here:
```
apt install libcurl4
```


Now, just to be safe, let’s back up the existing /etc/apt/sources.list file, because we’re going to replace its entire contents with this:
```
deb http://in.archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse
deb-src http://in.archive.ubuntu.com/ubuntu/ focal main restricted universe multiverse

deb http://in.archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://in.archive.ubuntu.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://in.archive.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://in.archive.ubuntu.com/ubuntu/ focal-security main restricted universe multiverse

deb http://in.archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://in.archive.ubuntu.com/ubuntu/ focal-backports main restricted universe multiverse

deb http://archive.canonical.com/ubuntu focal partner
deb-src http://archive.canonical.com/ubuntu focal partner
```

(Looking just now at that in.archive.ubuntu.com domain, I’m not sure if that’s a mirror in India and you could just use archive.ubuntu.com, or what. Just an observation. Do as you will.)

Once you’ve saved those changes, run these commands:
```
apt install –reinstall ubuntu-keyring
do-release-upgrade -c
do-release-upgrade
```

Again with the do-release-upgrade command you’ll need to follow the prompts, keeping your existing copies of config files when asked.

I’ve found that after running this, the server might not come back up on its own. I had to log into my Digital Ocean account after a few minutes and do a hard reboot, but once I did that, everything came back fine and I was on Ubuntu 20.04 LTS!

Verify:
```
## Current version ubuntu 16.04 LTS
root@am-testing:~# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 16.04.7 LTS
Release:        16.04
Codename:       xenial
root@am-testing:~# hostnamectl
   Static hostname: am-testing
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 82f1761a84e245aaadf3a009ff95af0b
           Boot ID: 9700f75b75aa4fd09b787bef3409c8c5
    Virtualization: kvm
  Operating System: Ubuntu 16.04.7 LTS
            Kernel: Linux 4.4.0-210-generic
      Architecture: x86-64

## After upgrade ubuntu 18.04 LTS
root@am-testing:~# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 18.04.6 LTS
Release:        18.04
Codename:       bionic
root@am-testing:~# hostnamectl
   Static hostname: am-testing
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 82f1761a84e245aaadf3a009ff95af0b
           Boot ID: 13bc74db55a04ca89d5ca0342fe22870
    Virtualization: kvm
  Operating System: Ubuntu 18.04.6 LTS
            Kernel: Linux 4.15.0-200-generic
      Architecture: x86-64

## After upgrade ubuntu 20.04 LTS 
root@am-testing:~# lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.5 LTS
Release:        20.04
Codename:       focal
root@am-testing:~# hostnamectl
   Static hostname: am-testing
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 82f1761a84e245aaadf3a009ff95af0b
           Boot ID: d617a983714a4a5887afe6bf2211a06c
    Virtualization: kvm
  Operating System: Ubuntu 20.04.5 LTS
            Kernel: Linux 5.4.0-135-generic
      Architecture: x86-64

```
