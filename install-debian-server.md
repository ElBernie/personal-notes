This document describes how to install a Debian server on VMWare, but can be used as a general guide for installing Debian on any hardware.

## Enable sudo mode

```bash
su
```

# Install basic packages

```bash
apt update &&
apt upgrade &&
apt install sudo git build-essential net-tools linux-headers-$(uname -r)
```

## Allow sudo mode for user

note: replace `USERNAME` with the real username

```bash
cd /etc &&
nano sudoers
```

Add the following line to the file, replacing `USERNAME` with the real username:

```bash

USERNAME ALL=(ALL) NOPASSWD: ALL
```

we can now exit the root shell:

```bash
exit
```
