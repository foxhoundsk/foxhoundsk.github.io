---
title:  "Linux kernel module development in Arch Linux based VM"
date: 2024-10-23T00:32:06+08:00
draft: false
description: Linux VM
isStarred: false
---

Linux kernel module (LKM) development is usually conducted inside a VM to prevent corruption of the dev box. Most of the LKM dev. setup guides are targeting Ubuntu based machine, but I love Arch Linux, and there have sparse resources regarding LKM dev. on Arch Linux based VM. So I would like to make a post discussing this. Specifically, I'll discuss how to make the kernel header that is required for out-of-date (no longer present in Pacman latest database) kernel images.

First, Linux requires that the kernel module has the same vermagic comparing to the targeting kernel, which means an appropriate kernel header is required for the module to load successfully. On Arch Linux, the **latest** header can be acquired by installing pakage `linux-headers`. However, due to the fact that the kernel came along with the prebuilt VM image, or the likes, can't usually match the latest header, we often times need to manually download the corresponding header [0].

To download specific kernel header, navigate to:

https://gitlab.archlinux.org/archlinux/packaging/packages/linux

Choose the git tag of interest and clone the repo, for example:
```
git clone --depth 1 --branch v6.10.8-arch1 https://github.com/archlinux/linux.git linux-6.10.8-arch1
```

Afterwards, run `makepkg -s` [1] to get the artifacts (e.g. Module.symvers) required for building the LKM.

During `makepkg`, one may encounter an error stating that the PGP key is unknown. To get rid of this, navigate to the header directory [2], then run:
```
gpg --import keys/pgp/*
```

In your device driver directory, make changes to the Makefile. Specifically, point `KERNEL_SRC` (or the likes) to the directory that you just ran `makepkg -s`. That is, the directory containing artifacts such as `Module.symvers`.

Now we are good to compile and load the kernel module for our specific kernel. Happy kernel hacking!

[0] One can of course update the kernel to the latest version to solve the version mismatch dilemma, but maybe one would like to stick with a specific kernel release.

[1] `-si` is also fine if one would like to have the header installed after the build.

[2] The directory containing directory `keys`.
