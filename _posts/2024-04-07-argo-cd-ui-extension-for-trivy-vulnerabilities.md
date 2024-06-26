---
layout: post
title: "Argo CD UI extension for Trivy Vulnerabilities"
description: "ArgoCD UI extension for visualizing and displaying Trivy report data"
category: "Security"
tags: argocd,trivy
comments: true
image: "/assets/img/vuln-dashboard.png"
---

After installing the [Trivy Operator](https://aquasecurity.github.io/trivy-operator), an open-source security scanner, into my Kubernetes cluster, I noticed the generated JSON vulnerability reports could get pretty big and would need some tool to parse them. 

My preference was to view the data in `Argo CD`, largely out of `Argo CD` being an *almost* default Kubernetes interface for cluster admins and developers.

Consequently, to vizualize and get better search over the `Trivy` vulnerabilities, I wrote an Argo CD UI extension, [argocd-trivy-extension](https://github.com/mziyabo/argocd-trivy-extension).

The UI extension comprises two `React` components, a searchable, sortable, grid and a dashboard made using [Grid.js](https://gridjs.io/) and [recharts](https://recharts.org) respectively. Vulnerability data is pulled into the components via the Argo CD API- This simplified the extension by avoiding using the Kubernetes API.

<img src="../../assets/img/vuln-table.png" width="85%"/> 


<img alt="vulnerabilities dashboard" src="../../assets/img/vuln-dashboard.png" width="85%"/>

Detailed installation instructions are in the project [README](https://github.com/mziyabo/argocd-trivy-extension) and involves loading the extension into the Argo CD server pods via an init container using the [argocd-extension-installer](https://github.com/argoproj-labs/argocd-extension-installer).

A quick but dirty test can be achieved by copying the extension via `kubectl cp` into the Argo CD servers `/tmp/extension/` path.

**TIP:** If you run into issues with installation check the init-container logs of the `argocd-extension-installer` e.g. if the extension does not render in the Argo CD UI. If it does render however, check the Dev Tools on your browser. 

What's next? Try the extension out and log an issue or contribute. This is in essence a quality of life extension for a security minded engineer to allow quick search, sorting and visualization of vulnerability data at the granular level of a Pod or ReplicaSet.