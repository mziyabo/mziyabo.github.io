---
layout: post
title: "Kubernetes charts and manifests with cdk8s"
description: "Kubernetes charts and manifests with Cdk8s"
category: 
tags: [cdk8s,Kubernetes,Typescript]
comments: true
---

[cdk8s](https://cdk8s.io/) is a great way to bundle and compose applications to be deployed to Kubernetes.

`cdk8s` uses a concept of an [APIObject](https://awscdk.io/packages/cdk8s@0.21.0#/./cdk8s.ApiObject) to create K8s api-resources in a programming language and for now this is Typescript and Python with more coming. Each `APIObject` is created in the context of a `Construct` and the constructs are composed together into a [Chart](https://awscdk.io/packages/cdk8s@0.21.0#/./cdk8s.Chart). The chart concept is ofcourse familiar and analogous to [helm](https://helm.sh/) charts- kind of.

An example to help make sense of this:

In Typescript you can declare a Construct that creates a Service `APIObject`/resource as follows:

``` typescript
class MyConstruct extends Construct {

constructor(scope:Construct, name:string) {
    super(scope, name)

new Service(this, 'service', {
    spec: {
        type: 'LoadBalancer',
        ports: [ { port: 80, targetPort: IntOrString.fromNumber(8080) } ],
        selector: label
    }}
   );
 }
}
```

From here `MyConstruct` can be instantiated in a `Chart` together with other Constructs that create their own resources.

To demonstrate composing the Constructs into a chart we can use say, [cdk8s-xray](https://www.npmjs.com/package/cdk8s-xray) and [cdk8s-redis](https://www.npmjs.com/package/cdk8s-redis), Constructs for [AWS X-Ray](https://aws.amazon.com/xray/) and `Redis`:

> You'd install cdk8s-redis and cdk8s-xray from **NPM**

``` typescript
import {Redis} from 'cdk8s-redis'
import { XRayApp, DaemonProtocol } from 'cdk8s-xray'

// inside your chart:
// add application constructs for X-Ray and Redis to chart

let xrayconfig = {
    image: "rnzdocker1/eks-workshop-x-ray-daemon:dbada4c77e6ae10ecf5a7b1c5864aa6522d9fb02",
    ns: "default",
    daemon: {
        daemonProtocol: DaemonProtocol.UDP,
        port: 2000,
        logLevel: "prod"
    }
}

new XRayApp(this, 'prod', xrayconfig);
new Redis(this, "redis", {
    labels:{
        "app":"redis"
    },
    slaveReplicas: 1
});
```

In the example, two separates Constructs, `XRayApp` and `Redis` are combined into a chart. Running `cdk8s synth` from the [cdk8s-cli](https://www.npmjs.com/package/cdk8s-cli) would generate a manifest with the Kubernetes api-resources defined in each Construct.

The advantages of the cdk8s approach include:

- Composability of multiple `Constructs` into Charts that can be deployed to a Kubernetes. 
- Additionally, as this is all just `Python` or `Typescript` you can add unit-tests to say check if every container in a DamoneSet has resource Limits defined- not easily done with YAML. 
- And if your're passing all this through a CI/CD workflow it can give some control over what get's deployed without deferring that to an K8s admission Controller or other Policy Engine which makes it a day one task.

It's a lot easier perhaps taking a look at a sample e.g. the [Github repo for cdk8s-xray](https://github.com/mziyabo/cdk8s-xray) (because I wrote it 👹). There are other great examples [here](https://github.com/dungahk/awesome-cdk8s) as well.

I hope to see `cdk8s` develop more- It's still early days but the potential is all there.