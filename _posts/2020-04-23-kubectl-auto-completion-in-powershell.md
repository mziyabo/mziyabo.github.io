---
layout: post
title: "PowerShell kubectl autocomplete"
description: "kubectl tab-completion in PowerShell"
category: 
tags: [PSKubectlCompletion,kubectl,PowerShell,auto-completion,tab-completion]
comments: true
---

⛔**update:** [PSKubectlCompletion](https://www.powershellgallery.com/packages/PSKubectlCompletion/) is no longer actively developed. [kubectl has added PowerShell completion](https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-pwsh/) which you can now use. Thank you to everybody who contributed and used the project 🖖🏽.

---
<br/>

[kubectl](https://kubernetes.io/docs/reference/kubectl/overview/) has been shipping command completers for bash and ZSH for a while now.  After taking a peek at the bash completion code I wrote a PowerShell port which I published as the [PSKubectlCompletion](https://www.powershellgallery.com/packages/PSKubectlCompletion/) Module in the `PSGallery`.

To install `PSKubectlCompletion` in PowerShell and register completions run the below commands:

{% highlight powershell %}
Import-Module -Name PSKubectlCompletion
Register-KubectlCompletion
{% endhighlight powershell %}

After registering, hitting the Tab key for kubectl commands should generate completions.

`PSKubectlCompletion` runs on PowerShell 5.1 and PowerShell Core v6.0 and higher. I had to upgrade [PSReadline](https://www.powershellgallery.com/packages/PSReadline/2.0.1) to v2.01 from beta on Linux to get it working just right.

The completion is done by binding `kubectl.exe` as a Native command in the [Register-ArgumentCompleter](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/register-argumentcompleter?view=powershell-7#outputs) cmdlet.

If you want to take a look at the code or manually install the Module- It's up on [GitHub](https://github.com/mziyabo/PSKubectlCompletion). Give it a try and hit me up if you want to share feedback or contribute. If you use the `awscli` and would like similar tab-completion checkout [AWSCompleter](https://www.powershellgallery.com/packages/AWSCompleter/) as well.