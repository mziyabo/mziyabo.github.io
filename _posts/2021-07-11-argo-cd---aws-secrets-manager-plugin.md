---
layout: post
title: "Argo CD AWS Secrets Manager plugin"
description: "An Argo CD plugin for AWS Secrets Manager"
category: 
tags: ["argocd","kubernetes","aws secrets manager","python"]
comments: true
---

**TLDR:** I wrote an Argo CD plugin that does dynamic placeholder replacement of secrets at deployment time from `AWS Secrets Manager`. Installation and usage are [on GitHub](https://github.com/mziyabo/argocd-aws-secret-plugin).

Now, a bit of background:

With [Argo CD](https://argoproj.github.io/argo-cd/) we can have Kubernetes manifests commited to version control and then have those manifest changes `synced` to a Kubernetes cluster. Effectively, we have Git as source of truth for K8s infrastructure and also Git driving the deployment of said infra. [A discussion on GitOps](https://about.gitlab.com/topics/gitops/) is an entire beast on it's own.

To avoid committing our secrets to version control we can hook into how `Argo CD` generates manifests and instead inject secrets dynamically at deployment time.

`Argo CD` provides a mechanism for building [custom plugins](https://argo-cd.readthedocs.io/en/stable/operator-manual/custom_tools/) to achive this. A plugin can be any program that writes out valid Kubernetes manifests to stdout- Simple, in a way, like CloudFormation Macros.

In this case, we read manifests from an `Argo CD` directory or helm application and then replace placeholder text in angle brackets e.g. `<PLACEHOLDER>` were **PLACEHOLDER** is an AWS Secret ID. 

A similar project exists for fetching secrets from [Vault Secrets Engine](https://github.com/IBM/argocd-vault-plugin) and was used to draw inspiration for the [AWS Secrets Manager plugin](https://github.com/mziyabo/argocd-aws-secret-plugin).

The code for our plugin is waaay simpler though. Essentially the below without boilerplate:

``` python
import re
import boto3
import yaml

def generate(template: str):

    pattern = r"(\<.*?\>)"
    client = boto3.client('secretsmanager')

    try:
        yaml_templates = yaml.safe_load_all(template)

        for yaml_template in yaml_templates:
            matches = re.findall(pattern, str(yaml_template))

            for match in matches:
                res = client.get_secret_value(
                    SecretId=re.sub("[<>]", "", match))
                if res['SecretString']:
                    template = template.replace(
                        match, res['SecretString'])

        print(template)
    except client.exceptions.from_code("ResourceNotExistsError") as rne:
        raise rne
    except Exception as e:
        raise e
```

The above simply matches a regex pattern for text (an AWS Secret Id) in angle brackets and replaces it (or injects as we've said) with an AWS Secret at deployment.

Usage and installation instructions of the plugin is in the [repository ReadMe](https://github.com/mziyabo/argocd-aws-secret-plugin).

My thought is a plugin should be a small piece of code doing a small bit of post template processing- Probably why I find it analogous to CloudFormation macros which are small in code-size typically. And this may also be because CloudFormation has to be singularily responsible for 90pct of all YAML processing on earth followed by Kubernetes 😝.

There are a few things I've left from the plugin code out depending on when you're reading this, but:

- It would be nice to conditionally ignore secret replacement based on an annotation in the manifest. This is pretty simple.
- Like [AWS CloudFormation dynamic parameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/dynamic-references.html#dynamic-references-secretsmanager), we could also replace only when we match text in angle brackets but also prefixed with `resolve:ssm:<parameter-name>:version` or `resolve:secretsmanager:<SecretID>:SecretString:password:EXAMPLE1-90ab-cdef-fedc-ba987EXAMPLE` for example.
- In the same vain, we can then fetch parameter values from `AWS SSM` and inject those into our Kubernetes manifests, just like we're doing with AWS Secrets Manager here.

In conclusion, the reasons for injecting at deployment time are to avoid committing senstive data to version control. This is something evident in GitOps approaches. To have an additional layer of control  through access policies, management and secret rotation around whatever it is we inject, we can then have this sort of post-template processing approach closer to deployment time as opposed to sitting statically in code.

Please do check out the [plugin](https://github.com/mziyabo/argocd-aws-secret-plugin) and feel welcome to contribute issues, thoughts or code.