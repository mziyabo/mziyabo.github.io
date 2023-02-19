---
layout: post
title: "Masking secret data in the Argo CD UI"
description: "Masking sensitive data in the Argo CD UI using a proxy"
category:
tags: ["argocd"]
comments: true
---

`Argo CD` has this peculiarity were sensitive data can sometimes get leaked out by the UI in *Live Manifests or DIFFs*. So even if you used something like [argocd-vault-plugin](https://github.com/argoproj-labs/argocd-vault-plugin) which injects a secret to replace a placeholder, the Argo CD generated manifests may eventually reveal the secret in some part of the UI.


To mask the data which shows up on Live Manifests and DIFFs, I wrote a masking proxy, [argocd-masking-proxy](https://github.com/mziyabo/argocd-masking-proxy), which does a regex match and replace on a pattern of the sensitive data we wanted to mask out with asterisks in the UI.

⚠️ **Disclaimer:** The proxy and project described here is experimental.

### Overview

The masking-proxy proxies a Kubernetes API server, in our case the cluster internal Kubernetes API endpoint `https://kubernetes.default.svc`. After receiving requests from Argo CD (e.g. for a Sync operation) it forwards those requests to the Kubernetes API server and runs a set of masking rules defined in a [proxy.conf.json](https://github.com/mziyabo/argocd-masking-proxy/blob/main/proxy.conf.json) which it then passes back to Argo CD. The rules are composed of a regex pattern to match the sensitive data and a replacement pattern.

### Installation

Installation is via Helm as below:

``` bash
## clone repo
git clone git@github.com:mziyabo/argocd-masking-proxy.git
cd masking-proxy

## install the masking-proxy helm chart
helm install --name masking-proxy ./chart/
```
The installation creates an Argo CD cluster which is visible from `Settings/Clusters` or via the Argo CD cli:

![cluster](/assets/img/cluster.png)

The masking rules are added via a ConfigMap which is also deployed in the Helm Chart. The properties in the ConfigMap are explained in the project [README](https://github.com/mziyabo/argocd-masking-proxy#configuration).

### Usage

Detailed usage is available from the [masking-proxy repository](https://github.com/mziyabo/argocd-masking-proxy#example). In short, the sample creates an Argo CD application which uses the masking-proxy address as the destination server:

``` bash
cat << EOF | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: sample
  namespace: argocd
spec:
  destination:
    namespace: sample
    server: https://masking-proxy.masking-proxy.svc.cluster.local
  project: default
  source:
    path: examples/app
    repoURL: https://github.com/mziyabo/masking-proxy
    targetRevision: HEAD
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
EOF
```

The sample application is composed of one resource, a ConfigMap with GitHub OAUTH client details as below.
```
apiVersion: v1
kind: ConfigMap
data:
   jcasc-default-config.yaml: |-
    ...
    securityRealm:
        github:
            clientID: "32f2133d7bd2d2215c6a"
            clientSecret: "e7da32a32a7ff720a3f007c07701777a2723c7c7"
```
Here the proxy matches and replaces these credentials using the following regex `(client(_?Secret|ID)):((\\s?\\\\+\"?)([a-z0-9]*)(\\\\\"|\"|\\s)?)` in the masking rules.

After syncing we will have the below:

**Live Manifest:**

![Live Manifest](/assets/img/livemanifest.png)

**DIFF:**

![DIFF](/assets/img/diff.png)

Almost, but not exactly what we wanted.

### Outcomes

The proxy masks out the *Live Manifest* data as seen in the screenshot on Lines 10,11,20,21 (compare this with the desired [sample template](https://github.com/mziyabo/argocd-masking-proxy/blob/main/examples/app/configmap.yaml)). However the DIFF still leaks out the credentials.

To address the DIFF leak we could later use [argocd-vault-plugin](https://github.com/argoproj-labs/argocd-vault-plugin) or similar, which would use a placeholder for the secret up until manifest deployment time in Argo CD.

One other outcome was that we'd created a persistent out-of-sync scenario where Argo CD views the asterisks from our masking operation as a deviation from the desired. That is fair, and we can use [Argo CD compare options](https://argo-cd.readthedocs.io/en/stable/user-guide/compare-options/) to ignore the ConfigMap in the application sync result.

### Conclusion

I think perhaps masking at the UI and not the Kubernetes API server level would be best moving forward. That could involve application middleware in the Argo CD server source for example.

The goose chase/rabbit-hole ends here.