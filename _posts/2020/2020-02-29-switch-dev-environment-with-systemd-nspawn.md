---
layout: post
title: Switch development environment with systemd-nspawn
author: HackingPheasant
categories: [guide]
tags: [switch, systemd, linux, chroot]
---
I've been wanting to compile Nintendo Switch homebrew and CFW (Custom Firmware) for a while now, but getting all the required software and dependencies setup (and maintained) on my Solus install seemed a tad to much effort for me at the time. As the year progressed I heard about systemd-nspawn during a discussion, and it was described to me as a *chroot on steroids*. Thinking back on wanting to compile Nintendo switch software and realising that systemd-nspawn may be the solution to this problem, I decided to give this a go.

The following intends to give you a quick overview of how I setup up the initially setup arch, the required dependiceis and gives a few quick examples of how to compile. This isn't intended to be a guide that will hand hold you through the process.

**Note:** This was done early 2019, so commands *and/or* instructions *an/may* differ from how you should be doing it now.

## Setup an arch systemd-nspawn container
To keep it simple, we will bootstrap arch and the final container will end up in `$HOME` (stuff like `/var/lib/machines` and the machinectl command are outside the scope of this blog post).
```bash
mkdir -p $HOME/arch-minimal-devkit64
wget 'ftp://ftp.fake.mirror/pub/archlinux/iso/latest/archlinux-bootstrap-*-x86_64.tar.gz' 
sudo tar -xzf archlinux-bootstrap-*-x86_64.tar.gz -C /tmp && rm archlinux-bootstrap-*-x86_64.tar.gz
```
Uncomment the relevant mirrors found at `/tmp/root.x86_64/etc/pacman.d/mirrorlist`.

Now lets bootstrap this container.

**Note:** At the mount command, I had used `/dev/nvme0n1p5`, I suggest you replace this with the correct location pretaining to your setup. I was lazy and just mounted `/` and used my home directory in the pacstrap command. Be careful of what you `rm` when `/` is still mounted, it may cause potential data loss.

```bash
sudo /tmp/root.x86_64/bin/arch-chroot /tmp/root.x86_64/
pacman-key --init
pacman-key --populate archlinux
mount /dev/nvme0n1p5 /mnt
pacstrap -i -c /mnt/home/hackingpheasant/arch-minimal-devkit64/ base base-devel --ignore linux --cachedir /mnt/tmp
exit
```

Now lets do a full system boot into our brand new arch systemd-nspawn container
```bash
sudo systemd-nspawn -b -D $HOME/arch-minimal-devkit64/
```
**Flags used:**
* `-b` *Boot up full system (i.e. invoke init)*
* `-D` *Root directory for the container*


You should be greeted with the following
```bash
Spawning container arch-minimal-devkit64 on /home/hackingpheasant/arch-minimal-devkit64.
Press ^] three times within 1s to kill container.
systemd 239 running in system mode. (+PAM +AUDIT -SELINUX -IMA -APPARMOR +SMACK -SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD +IDN2 -IDN +PCRE2 default-hierarchy=hybrid)
Detected virtualization systemd-nspawn.
Detected architecture x86-64.

Welcome to Arch Linux!

File /usr/lib/systemd/system/systemd-journald.service:36 configures an IP firewall (IPAddressDeny=any), but the local system does not support BPF/cgroup based firewalling.
Proceeding WITHOUT firewalling in effect! (This warning is only shown for the first loaded unit using IP firewalling.)
[  OK  ] Listening on LVM2 poll daemon socket.
[  OK  ] Created slice system-getty.slice.
[  OK  ] Listening on Process Core Dump Socket.
[  OK  ] Listening on Device-mapper event daemon FIFOs.
[  OK  ] Reached target Remote File Systems.
[  OK  ] Reached target Swap.
[  OK  ] Listening on initctl Compatibility Named Pipe.
[  OK  ] Started Dispatch Password Requests to Console Directory Watch.
[  OK  ] Created slice User and Session Slice.
[  OK  ] Listening on Journal Socket.
         Mounting POSIX Message Queue File System...
         Mounting FUSE Control File System...
         Mounting Huge Pages File System...
[  OK  ] Listening on LVM2 metadata daemon socket.
         Starting Monitoring of LVM2 mirrors…ng dmeventd or progress polling...
[  OK  ] Started Forward Password Requests to Wall Directory Watch.
[  OK  ] Reached target Paths.
[  OK  ] Reached target Local Encrypted Volumes.
[  OK  ] Reached target Slices.
[  OK  ] Listening on Journal Socket (/dev/log).
         Starting Journal Service...
         Starting Remount Root and Kernel File Systems...
[  OK  ] Mounted POSIX Message Queue File System.
[  OK  ] Mounted FUSE Control File System.
[  OK  ] Mounted Huge Pages File System.
[  OK  ] Started LVM2 metadata daemon.
[  OK  ] Started Remount Root and Kernel File Systems.
         Starting Create System Users...
[  OK  ] Started Monitoring of LVM2 mirrors,…sing dmeventd or progress polling.
[  OK  ] Started Journal Service.
         Starting Flush Journal to Persistent Storage...
[  OK  ] Started Flush Journal to Persistent Storage.
[  OK  ] Started Create System Users.
[  OK  ] Reached target Local File Systems (Pre).
[  OK  ] Reached target Local File Systems.
         Starting Rebuild Dynamic Linker Cache...
         Starting Rebuild Journal Catalog...
         Starting Create Volatile Files and Directories...
[  OK  ] Started Rebuild Journal Catalog.
[  OK  ] Started Create Volatile Files and Directories.
         Starting Update UTMP about System Boot/Shutdown...
[  OK  ] Started Update UTMP about System Boot/Shutdown.
[  OK  ] Started Rebuild Dynamic Linker Cache.
         Starting Update is Completed...
[  OK  ] Started Update is Completed.
[  OK  ] Reached target System Initialization.
[  OK  ] Started Daily Cleanup of Temporary Directories.
[  OK  ] Listening on D-Bus System Message Bus Socket.
[  OK  ] Reached target Sockets.
[  OK  ] Reached target Basic System.
         Starting Permit User Sessions...
[  OK  ] Started Daily verification of password and group files.
[  OK  ] Started Daily man-db cache update.
[  OK  ] Started D-Bus System Message Bus.
         Starting Login Service...
[  OK  ] Started Daily rotation of log files.
[  OK  ] Reached target Timers.
[  OK  ] Started Permit User Sessions.
[  OK  ] Started Console Getty.
[  OK  ] Reached target Login Prompts.
[  OK  ] Started Login Service.
[  OK  ] Reached target Multi-User System.
[  OK  ] Reached target Graphical Interface.

Arch Linux 4.18.9-92.current (console)

arch-minimal-devkit64 login: 
```
## Setting up users and installing dependencies
Lets login, setup a user that isn't root and install some quality of life programs.

```bash
Arch Linux 4.18.9-92.current (console)

arch-minimal-devkit64 login: root
[root@arch-minimal-devkit64 ~]# pwd
/root
[root@arch-minimal-devkit64 ~]# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime 
[root@arch-minimal-devkit64 ~]# useradd --create-home switchdev
[root@arch-minimal-devkit64 ~]# passwd switchdev
New password: 
Retype new password: 
passwd: password updated successfully

[root@arch-minimal-devkit64 ~]# visudo
```
You will need to find the lines in the file that grant sudo access to users in the group wheel when enabled. Below is what you should be looking for.

```bash
## Allows people in group wheel to run all commands
# %wheel        ALL=(ALL)       ALL
```

Remove the comment character (#) at the start of the second line. This enables the configuration option.

```bash
[root@arch-minimal-devkit64 ~]# usermod -aG wheel switchdev
[root@arch-minimal-devkit64 ~]# pacman -Syu
[root@arch-minimal-devkit64 ~]# pacman -S linux-tools vim lib32-gcc-libs ed python-setuptools python-pip git zip libyaml
[root@arch-minimal-devkit64 ~]# exit
logout
```
**Note:** Best to edit `/etc/pacman.conf` so the *IgnorePkg* line looks like so `IgnorePkg   = linux`, before installing new packages otherwise you will have to pass `--ignore linux` on your pacman command each time you do a system update.

Now that we have an account that isn't root, lets log into it and disable root account access while we are at it.

```bash
Arch Linux 4.18.9-92.current (console)

arch-minimal-devkit64 login: switchdev 
Password: 
[switchdev@arch-minimal-devkit64 ~]$ 
[switchdev@arch-minimal-devkit64 ~]$ pwd
/home/switchdev
[switchdev@arch-minimal-devkit64 ~]$ sudo passwd -l root # Disable root account
```

Lets install the dependencies needed to compile a majority of switch homebrew.

**Note:** The following is copied from devkitPro pacman [page](https://devkitpro.org/wiki/devkitPro_pacman), so please visit there for the latest instructions

We first import the key which is used to validate the packages

```bash
[switchdev@arch-minimal-devkit64 ~]$ sudo pacman-key --recv F7FD5492264BB9D0
[switchdev@arch-minimal-devkit64 ~]$ sudo pacman-key --lsign F7FD5492264BB9D0
[switchdev@arch-minimal-devkit64 ~]$ sudo pacman -U https://downloads.devkitpro.org/devkitpro-keyring-r1.787e015-2-any.pkg.tar.xz

```
Now we add the devkitPro repositories.

**Note:** two repositories will be added - one for the libraries and another for the host specific tools.

```bash
[switchdev@arch-minimal-devkit64 ~]$ sudo vim /etc/pacman.conf
```
Now add the following lines to the end of `/etc/pacman.conf`

```bash
[dkp-libs]
Server = http://downloads.devkitpro.org/packages
   
[dkp-linux]
Server = http://downloads.devkitpro.org/packages/linux
```
And then we make sure the package list is update
```bash
[switchdev@arch-minimal-devkit64 ~]$ sudo pacman -Syu 
```


Now lets install some packages so we can compile atmosphere, nx-hbloader, nx-hbmenu, switch-examples, Goldleaf and checkpoint.
```bash
[switchdev@arch-minimal-devkit64 ~]$ sudo pacman -S switch-dev switch-portlibs devkitARM switch-freetype switch-mpg123 switch-glad
[switchdev@arch-minimal-devkit64 ~]$ source /etc/profile.d/devkit-env.sh # Or just log out and back in.
[switchdev@arch-minimal-devkit64 ~]$ sudo pip install pyhash pycrypto pycryptoplus
```

We are going to remove the libnx provided by devkitpro, as compiling atmosphere requires the bleeding edge libnx binary's.We will remove the binary's installed by pacman and instead will download, compile and install the git version.

```bash
[switchdev@arch-minimal-devkit64 ~]$ mkdir Switch && cd Switch/
[switchdev@arch-minimal-devkit64 Switch]$ sudo rm -rf /opt/devkitpro/libnx/
[switchdev@arch-minimal-devkit64 Switch]$ git clone https://github.com/switchbrew/libnx.git
[switchdev@arch-minimal-devkit64 Switch]$ cd libnx/
[switchdev@arch-minimal-devkit64 libnx]$ make
[switchdev@arch-minimal-devkit64 libnx]$ sudo -E bash -c 'make install' # https://stackoverflow.com/q/8633461/4634499
make -C nx/ install
make[1]: Entering directory '/home/switchdev/Switch/libnx/nx'
make[2]: '/home/switchdev/Switch/libnx/nx/lib/libnx.a' is up to date.
make[2]: '/home/switchdev/Switch/libnx/nx/lib/libnxd.a' is up to date.
mkdir -p /opt/devkitpro/libnx
bzip2 -cd libnx-1.4.1.tar.bz2 | tar -xf - -C /opt/devkitpro/libnx
make[1]: Leaving directory '/home/switchdev/Switch/libnx/nx'
[switchdev@arch-minimal-devkit64 libnx]$ sudo chown -R root:root /opt/devkitpro/libnx/
[switchdev@arch-minimal-devkit64 libnx]$ cd ../
```
**Note:** Best to edit `/etc/pacman.conf` so the *IgnorePkg* line looks like so `IgnorePkg   = linux libnx` to stop pacman complaining.

## Compiling Nintendo Switch custom firmware and homebrew

Lets compile the open source and free Nintendo Switch CFW known as Atmosphere.

**Note:** Since the TSEC root keys needed to sign sept aren't publicly shared(sharing them is illegal), we will have to borrow `sept-secondary_00.enc` and `sept-secondary_01.enc` from public atmosphere releases. They can be found at [here](https://github.com/Atmosphere-NX/Atmosphere/releases)
 
**Note 2:** `make` builds atmosphere's components, but doesn't package them up. `make dist` builds a release into the "out" folder.
The releases on github are just the output of `make dist`.

```bash
[switchdev@arch-minimal-devkit64 Switch]$ git clone https://github.com/Atmosphere-NX/Atmosphere.git
[switchdev@arch-minimal-devkit64 Switch]$ cd Atmosphere/
[switchdev@arch-minimal-devkit64 Atmosphere]$ make dist SEPT_00_ENC_PATH=$(sept-secondary_00.enc) SEPT_01_ENC_PATH=$(sept-secondary_01.enc)
[switchdev@arch-minimal-devkit64 Atmosphere]$ ls out/
atmosphere-0.7.1.zip  fusee-primary.bin
[switchdev@arch-minimal-devkit64 Atmosphere]$ cd ..
```

Now lets compile some Nintendo Switch Homebrew

```bash
[switchdev@arch-minimal-devkit64 switch]$ git clone https://github.com/switchbrew/switch-examples
[switchdev@arch-minimal-devkit64 switch]$ cd switch-examples/
[switchdev@arch-minimal-devkit64 switch-examples]$ make
[switchdev@arch-minimal-devkit64 switch-examples]$ cd ..
[switchdev@arch-minimal-devkit64 Switch]$ git clone https://github.com/switchbrew/nx-hbloader.git
[switchdev@arch-minimal-devkit64 Switch]$ cd nx-hbloader/
[switchdev@arch-minimal-devkit64 nx-hbloader]$ make
[switchdev@arch-minimal-devkit64 nx-hbloader]$ cd ..
[switchdev@arch-minimal-devkit64 Switch]$ git clone https://github.com/switchbrew/nx-hbmenu.git
[switchdev@arch-minimal-devkit64 Switch]$ cd nx-hbmenu/
[switchdev@arch-minimal-devkit64 nx-hbmenu]$ make nx
[switchdev@arch-minimal-devkit64 nx-hbmenu]$ cd ..
[switchdev@arch-minimal-devkit64 Switch]$ git clone https://github.com/XorTroll/Goldleaf.git
[switchdev@arch-minimal-devkit64 Switch]$ cd Goldleaf/
[switchdev@arch-minimal-devkit64 Goldleaf]$ make
[switchdev@arch-minimal-devkit64 Goldleaf]$ cd ..
[switchdev@arch-minimal-devkit64 Switch]$ git clone https://github.com/FlagBrew/Checkpoint.git
[switchdev@arch-minimal-devkit64 Switch]$ cd Checkpoint/
[switchdev@arch-minimal-devkit64 Checkpoint]$ make
```

Throw the CFW and Homebrew you just compiled onto your hackable firmware and give them a test run.

## Final Words

I wanted to compile Atmosphere for my Nintendo Switch and ended up learning a little bit about systemd-nspawn in the process, hopefully you find this knowledge as useful as I did, even if what I wrote here was terse and to the point. 
