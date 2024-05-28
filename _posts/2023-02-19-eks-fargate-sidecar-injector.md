---
layout: post
title: "EKS Fargate sidecar injection using mutating webhooks"
description: "Using Kubernetes mutating webhooks to perform sidecar injection on Fargate pods"
category: 
tags: ["aws","fargate","eks","mutating webhook","kubernetes"]
comments: true
---

EKS Fargate schedules only one pod per node. This means you can't run [Daemonsets](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) on the same node and have to opt for a sidecar pattern to deploy the Daemonsets intended functionality.

Handling the sidecar creation can be done via [Mutating Webhooks](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#mutatingadmissionwebhook). The advantage is that we can keep our Kubernetes templates simpler and defer the decision to add the Fargate sidecars until the pods are actually ready to be scheduled- As opposed to baking it into those templates at design-time.

In order to address generic sidecar injection onto Fargate tasks I wrote a mutating webhook published to [ArtifactHub](https://artifacthub.io/packages/helm/mziyabo/fargate-sidecar-injector). The code is also on [GitHub](https://github.com/mziyabo/fargate-eks-sidecar-injector), and you can check that out.

The mutating webhook works by checking the pod for the fargate pod annotation; `eks.amazonaws.com/fargate-profile`. This annotation would have been injected into the pod by the `0500-amazon-eks-fargate-mutation.amazonaws.com` mutating webhook.

From here, we fetch the sidecars containers and volumes from a ConfigMap and patch the Fargate pods accordingly.

To install the webhook use the Helm chart as below:

```bash
helm repo add mziyabo https://mziyabo.github.io/helm-charts
helm install fargate-sidecar-injector mziyabo/fargate-sidecar-injector
```

After installation, add a ConfigMap named `fargate-injector-sidecar-config` in the webhook namespace with the below format:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fargate-injector-sidecar-config
data: 
  default: |-  # <- Fargate Profile
    spec:
      volumes:
      - name: log
        emptyDir: {}
      containers:
      - image: busybox
        name: sidecar
        args:
        - /bin/sh
        - -c
        - >
          while true; do
            echo "$(date) INFO hello from main-container" >> /var/log/myapp.log ;
            sleep 1;
          done
        volumeMounts:
        - name: log
          mountPath: /var/log
```

In the above example, we're targeting the fargate-profile named `default` - Meaning we only inject the sidecars into that specific profile. To inject them into a different profile we'd have to add it to the ConfigMap as well.

This is to have more fine-grained control but can also be tedious to maintain some global settings across all fargate-profiles. This is something I'll potentially address in future.

To test the config, you can run a pod with matching fargate-profile selectors e.g.:

```bash
kubectl run test -it --rm -n default --image curlimages/curl --pod-running-timeout=2m -- sh
```

> Note: To cleanup, you will have to run `kubectl delete mutatingwebhookconfiguration fargate-sidecar-injector` after `helm uninstall` since the **MutatingWebhookConfiguration** is installed using a helm post-install hook. [Helm docs explain the limitation](https://helm.sh/docs/topics/charts_hooks/#hook-resources-are-not-managed-with-corresponding-releases).

Ultimately, what we want to achieve is a single set of Kubernetes templates (across EC2/Fargate hosts) that don't have to contain logic for the sidecar being conditionally injected on Fargate. This also provides a central way to manage the sidecars, especially if you have many helm charts that need the same set of sidecars.

The [Daemonsets on Fargate issue](https://github.com/aws/containers-roadmap/issues/971) remains open on GitHub. This solution attempts to bridge the gap while AWS figure out how to make it work. 

Feel free to contribute and test it out.

