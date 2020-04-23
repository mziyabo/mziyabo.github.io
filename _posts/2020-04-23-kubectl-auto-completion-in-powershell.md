---
layout: post
title: "PowerShell tab-completion for kubectl"
description: "kubectl tab-completion in PowerShell"
category: 
tags: [PSKubectlCompletion,kubectl,PowerShell,auto-completion,tab-completion]
comments: true
---

[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) has been shipping command completers for bash and ZSH for a while now. After taking a peek at the bash completions, I wrote a port of the code in PowerShell and published the [PSKubectlCompletion](https://www.powershellgallery.com/packages/PSKubectlCompletion/) Module in the `PSGallery`.

To install `PSKubectlCompletion` in PowerShell and register completions run the below commands:

{% highlight PowerShell  %}
Import-Module -Name PSKubectlCompletion
Register-KubectlCompletion
{% endhighlight PowerShell %}

After registering, hitting the Tab key for kubectl commands should generate completions.

`PSKubectlCompletion` runs on PowerShell 5.1 and PowerShell Core v6.0 and higher. I had to upgrade [PSReadline](https://www.powershellgallery.com/packages/PSReadline/2.0.1) to v2.01 from beta on Linux to get it working just right.

The Module borrows from the bash completions but not entirely- In particular, I found `ArrayLists` easier to use as collections for flags and commands. Also, the completion itself is done by binding `kubectl.exe` as a Native command using the [Register-ArgumentCompleter](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/register-argumentcompleter?view=powershell-7#outputs) cmdlet.

If you want to take a look at the code or manually install the Module- It's up on the [PSKubectlCompletion repository on GitHub](https://github.com/mziyabo/PSKubectlCompletion).

Do install the auto-completion [Module](https://www.powershellgallery.com/packages/PSKubectlCompletion/), try it out, and hit me up if you want to share feedback or contribute.