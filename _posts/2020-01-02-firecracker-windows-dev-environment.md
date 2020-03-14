---
title: "Building Firecracker in WSL2"
description: ""
category: "Firecracker"
tags: [Firecracker, Rust]
comments: true
---

[Firecracker](https://github.com/firecracker-microvm/firecracker) is a virtual machine monitor used to create micro-vms.

In an attempt to learn it- but mostly learn [Rust](https://www.rust-lang.org/), I cloned the project. However it turned out the Firecracker [build scripts](https://github.com/firecracker-microvm/firecracker/blob/master/tools/devtool) are written as a bash script that I failed to port to PowerShell. The build scripts also spun up a Docker build container along the way.

The quick and dirty workaround to getting the builds running was to enable [WSL 2](https://docs.microsoft.com/en-us/windows/WSL/WSL2-install) which shipped in Windows 10 builds 18917 or higher. Dirty since you need to sign up to the Windows Insider preview at the time of writing.

WSL2 could spin up the docker build container (not possible in WSL 1) while allowing me to execute the rest of the build scripts which was written in unportable bash code.

Building is as far as you can go though on Windows. To run Firecracker you need KVM and WSL2 doesn't allow you to run KVM just yet as it doesn't support nested virtualization. To verify this I ran Firecracker's `/tools/devtools checkenv` command in WSL:

![/tools/checkenv](/assets/posts/checkenv.PNG)

I also ran [kvm-ok](http://manpages.ubuntu.com/manpages/xenial/man1/kvm-ok.1.html) which makes multiple KVM checks :). In the WSL2 case the checks fail because VT CPU flags, i.e. `vmx` for **Intel** and/ `svm` for **AMD** are not present in WSL2s Kernel.

``` sh
# grepping /proc/cpuinfo for VT flags
egrep -m1 -w '^flags[[:blank:]]*:' /proc/cpuinfo | egrep -wo '(vmx|svm)'
```

A [GitHub issue](https://github.com/microsoft/WSL/issues/4193) to add nested virtualization/ enable these flags in WSL exists. The suggested workaround is to build a WSL Kernel with KVM enabled.. in case you want to go down a rabbit🐰 hole.

For more on setting up Firecracker dev-boxes on the more commonly used Linux and MacOS [read here](https://github.com/firecracker-microvm/firecracker/blob/777e366329c48764db51325eda241943ca485d97/docs/dev-machine-setup.md).