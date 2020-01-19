---
title: "Firecracker Windows Dev-Environment"
description: ""
category: "Firecracker"
tags: [Firecracker, Rust]
comments: true
---

[Firecracker](https://github.com/firecracker-microvm/firecracker) is a virtual machine monitor used to create micro-vms.

In an attempt to learn it and also learn [Rust](https://www.rust-lang.org/) I cloned the project however it turned out the Firecracker [build scripts](https://github.com/firecracker-microvm/firecracker/blob/master/tools/devtool) assume you're working on a Linux dev box. Fortunately the builds are dockerized meaning Windows could be used as a dev environment.

The quick and dirty workaround to getting the builds running (for me) was to enable [WSL 2](https://docs.microsoft.com/en-us/windows/WSL/WSL2-install) shipped in Windows 10 builds 18917 or higher. Dirty since you need to sign up to the Windows Insider preview at the time of writing.

WSL 2 allows spinning up the docker build container- (previously not possible in WSL 1) while allowing us to execute the build scripts which are written in bash.

On Windows, building is as far as you can go though- to run the Firecracker binary you need KVM and WSL 2 doesn't allow you to run KVM just yet as it doesn't support nested virtualization. 
To verify this you can run [kvm-ok](http://manpages.ubuntu.com/manpages/xenial/man1/kvm-ok.1.html) or Firecracker's `/tools/devtools checkenv` command in WSL:

![/tools/checkenv](/assets/posts/checkenv.PNG)

`kvm-ok` makes multiple checks- and in the WSL 2 case the checks fail because virtualization technology cpu flags `vmx` for **Intel** and/ `svm` for **AMD** are not present in WSL 2.

``` sh
# grepping /proc/cpuinfo for VT flags
egrep -m1 -w '^flags[[:blank:]]*:' /proc/cpuinfo | egrep -wo '(vmx|svm)'
```

A [GitHub issue](https://github.com/microsoft/WSL/issues/4193) exists to add nested virtualization/ enable these flags in WSL. A suggested workaround is to build a WSL Kernel with KVM enabled.. in case you want to go down the rabbit🐰 hole.

For more on setting up Firecracker dev-boxes on Linux and MacOS [read here](https://github.com/firecracker-microvm/firecracker/blob/777e366329c48764db51325eda241943ca485d97/docs/dev-machine-setup.md).