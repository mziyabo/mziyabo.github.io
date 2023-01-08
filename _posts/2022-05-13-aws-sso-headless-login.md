---
layout: post
title: "AWS SSO headless login"
description: "AWS SSO headless signin from awscli"
category: ["AWS SSO"]
tags: ["AWS SSO","headless-sso","awscli", "golang"]
comments: true
---

[AWS Single Sign-On (AWS SSO)](https://docs.aws.amazon.com/singlesignon/latest/userguide/what-is.html) allows SSO users to sign-in using the awscli via the `aws sso login` command. As part of the typical login flow, a browser (default browser, unless `BROWSER` environment variable  exists) is launched for the user to perform a number of verification and authorization steps.

This [neatly illustrated sequence diagram](https://twitter.com/ben11kehoe/status/1260646764966670337) explaining the login flow provides a good overview of the process.

One of the switches to the [login command](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sso/login.html) is `--no-browser` which prints the AWS SSO verification url to `stdout`, without opening a browser, and then polls for verification success before timing out if none is received.

To perform a headless login you'd have to control the login flow from the verification url to completion using a browser automation library - Which is what we did using [go-rod](https://go-rod.github.io/#/).

The code of our headless login implementation is available at this [github.com/mziyabo/headless-sso](https://github.com/mziyabo/headless-sso) and can be installed via: 
``` bash
go install github.com/mziyabo/headless-sso@latest
```

`headless-sso` parses the output of the aws sso login command and greps out the verification url which it then controls using `go-rod`

Usage: 
``` bash
aws sso login  --profile <aws profile> --no-browser | headless-sso
```

We decided to use `netrc` to store the AWS Console credentials to shave time off entering credentials in the terminal. We also cache the AWS SSO cookies which we reuse to make subsequent sign-ins faster. Note the awscli already caches the cookies under `~/.aws/sso/cache/` but we maintained a separate location.

## Limitations

The only supported flow of the program is AWS SSO using hardware MFA (e.g. yubikey) - Therefore it can't take MFA input from the cli nor can it skip MFA. 

The limitation on not having MFA code input is because initially the usage of the command assumed we'd have access to `stdin` after reading out the verification url. In reality it is not available because soon after the url is generated, the `aws sso login` command starts polling for verification success and doesn't close the write end of the pipe. Effectively you can't read from the terminal until polling is done 🤦🏾‍♂️.

The solution is to perhaps not read the verification url from the `aws sso login` command and generate this url from headless-sso itself via the [AWS SSO-OIDC](https://docs.aws.amazon.com/singlesignon/latest/OIDCAPIReference/API_Operations.html) API actions.

I also considered creating a new tty for stdin to read from but that wouldn't be supported on Windows.

Last, I didn't have any reason to support the no-MFA use-case - This is not available to me in my daily usage of AWS SSO and not really recommended.

## Conclusions

The initial motivation was to avoid switching programs from terminal to browser and clicking through a bunch of windows. Also we'd hoped to shave time off the login process. 

I think we achieved the first objective, the second, not so clear - It takes 30s on average to go through this for me when using [headless-sso](https://github.com/mziyabo/headless-sso).

In the end, I leave this here for whomever wants to walk this path to feel for the stones when crossing the river. 
