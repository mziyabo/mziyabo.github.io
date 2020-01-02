---
layout: post
title: "Building Firecracker on Windows"
description: ""
category: "Firecracker"
tags: []
---

[Firecracker](https://github.com/firecracker-microvm/firecracker) is a virtual machine monitor used to create micro-vms.

Currently, the Firecracker builds are dockerized however the firecracker [build scripts](https://github.com/firecracker-microvm/firecracker/blob/master/tools/devtool) assume you're working on a Linux dev box.

To run builds on a Windows development environment the workaround is to enable [WSL 2](https://docs.microsoft.com/en-us/windows/WSL/WSL2-install) shipped in Windows 10 builds 18917 or higher.

WSL 2 allows spinning up the docker build container- (previously not possible in WSL 1) while allowing us to run the build commands which are in a bash script.

To run the Firecracker binary you need KVM and WSL 2 doesn't allow you to run KVM just yet as it doesn't support nested virtualization. 
To verify this you can run [kvm-ok](http://manpages.ubuntu.com/manpages/xenial/man1/kvm-ok.1.html) or Firecracker's `/tools/devtools checkenv` command:

![/tools/checkenv](/assets/posts/checkenv.PNG)

`kvm-ok` makes multiple checks- and in this case the checks fail because virtualization technology cpu flags `vmx` for **Intel** and/ `svm` for **AMD** are not present in WSL 2.

``` sh
# grepping /proc/cpuinfo for VT flags
egrep -m1 -w '^flags[[:blank:]]*:' /proc/cpuinfo | egrep -wo '(vmx|svm)'
```

A [GitHub issue](https://github.com/microsoft/WSL/issues/4193) on the WSL repo exists to add nested virtualization/ enable these flags. A potential workaround is to build a Microsoft WSL Kernel with kvm enabled in case you want to go down the rabbit hole 🐰.

For more on setting up Firecracker dev-boxes on Linux and MacOS [read here](https://github.com/firecracker-microvm/firecracker/blob/777e366329c48764db51325eda241943ca485d97/docs/dev-machine-setup.md).