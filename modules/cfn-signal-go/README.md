**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-asg>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-asg/blob/master/modules/cfn-signal-go/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# CFN Signal Go

This module implements the [AWS CloudFormation cfn-signal
script](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-signal.html) in Go. The original `cfn-signal`
script is implemented in Python 2.x, which is not available in all environments (especially those that have Python 3.x
installed). Therefore, we have re-implemented the script in Go, which makes it easy to create cross-platform binaries
called `cfn-signal-go`.

## Why are we using CloudFormation?

Although we typically use [Terraform](https://www.terraform.io/) to provision infrastructure, Terraform does not
support rolling deployment for Auto Scaling Groups (ASG). As a workaround, the [asg-rolling-deploy-dynamic
module](/modules/asg-rolling-deploy-dynamic) creates the ASG using
[CloudFormation](https://aws.amazon.com/cloudformation/), which does support rolling deployment for ASGs. In order for
this rolling deployment to deploy your EC2 Instances, you need to signal CloudFormation when each EC2 Instance is up
and running. This is normally done using the Python `cfn-signal` script, but in environments where that script does not
work, you can use the `cfn-signal-go` binaries instead.

## How do you install cfn-signal-go?

You can install `cfn-signal-go` using the [Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer):

```bash
gruntwork-install --binary-name "cfn-signal-go" --repo "https://github.com/gruntwork-io/module-asg" --tag "0.0.23"
```

Alternatively, you can download the `cfn-signal-go` binary for your OS on the [releases
page](https://github.com/gruntwork-io/module-asg-public/releases).

## How do you run cfn-signal-go?

You'll typically want to run `cfn-signal-go` as part of [User
Data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html#user-data-shell-scripts) once your server is
running and ready to serve requests. Provide a User Data script that looks like the following:

```bash
#!/bin/bash
cfn-signal-go --stack-name MyAsgStack --logical-resource-id MyAsg --status SUCCESS
```

Note that signaling CloudFormation requires that your EC2 Instance has an IAM role with `cloudformation:SignalResource`
permissions. The [asg-rolling-deploy-dynamic module](/modules/asg-rolling-deploy-dynamic) creates these permissions for
you automatically.

## Is there an alternative to using cfn-signal-go?

If Python works on your AMI, you can use the [cloudformation-scripts module](/modules/cloudformation-scripts).

