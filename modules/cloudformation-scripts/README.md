**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-asg>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-asg/blob/master/modules/cloudformation-scripts/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# CloudFormation Scripts

You can use this module to install the [CloudFormation Helper
Scripts](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-helper-scripts-reference.html) on your
EC2 Instances. These scripts can be used to signal CloudFormation when an EC2 Instance is up and running so
CloudFormation knows if a rolling deployment worked or needs to be rolled back.

These scripts depend on Python 2.x. If you cannot install Python on your EC2 Instances, consider using the standalone
Go binary from the [cfn-signal-go module](/modules/cfn-signal-go).

## Why are we using CloudFormation?

Although we typically use [Terraform](https://www.terraform.io/) to provision infrastructure, Terraform does not
support rolling deployment for Auto Scaling Groups (ASG). As a workaround, the [asg-rolling-deploy-dynamic
module](/modules/asg-rolling-deploy-dynamic) creates the ASG using 
[CloudFormation](https://aws.amazon.com/cloudformation/), which does support rolling deployment for ASGs. If you're 
using the asg-rolling-deploy-dynamic module, you'll need to install the CloudFormation Helper Scripts on your EC2 
Instances using this module.

## How do you install the CloudFormation Helper Scripts on your EC2 Instances?

If you're using Amazon Linux, the scripts are already installed. For other Linux distros, you just need to run
`install-cloudformation-scripts.sh`.

Note that this can be handled for you automatically using the
[Gruntwork Installer](https://github.com/gruntwork-io/gruntwork-installer):

```bash
gruntwork-install --module-name `cloudformation-scripts` --repo https://github.com/gruntwork-io/module-asg
```

The best way to do that is in a [Packer](https://www.packer.io/) template. Check out the [asg-rolling-deploy-dynamic
example](/examples/asg-rolling-deploy-dynamic) for details.

## What do you do after the scripts are installed?

Once the scripts are installed, you'll want to run the `cfn-signal` script as part of [User
Data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html#user-data-shell-scripts) once your server is
running and ready to serve requests.

User Data scripts are executed as the root user, so you need to specify the absolute path for `cfn-signal`. On Amazon
Linux, it will be:

```bash
#!/bin/bash
/opt/aws/bin/cfn-signal --success true --resource MyAsg --stack MyAsgStack
```

On other flavors of Linux, as installed by the scripts in this module, it will be:

```bash
#!/bin/bash
/usr/local/bin/cfn-signal --success true --resource MyAsg --stack MyAsgStack
```

See the [cfn-signal documentation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-signal.html) for
details.

## Is there an alternative to using these scripts?

One alternative is to use a standalone binary compiled from the [cfn-signal-go module](/modules/cfn-signal-go).

Another alternative is to use the [SignalResource
API](http://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_SignalResource.html). This API requires
correctly formatting and signing requests, so your best bet is to use the [AWS CLI](https://aws.amazon.com/cli/) to do
the work for you. Here's an example of using the AWS CLI on an EC2 Instance to signal CloudFormation:

```bash
#!/bin/bash

AWS_DEFAULT_REGION="us-east-1"
INSTANCE_ID=$(curl http://169.254.169.254/latest/meta-data/instance-id)

aws cloudformation signal-resource --stack-name MyAsgStack --logical-resource-id MyAsg --unique-id "$INSTANCE_ID" --status SUCCESS
```

Note that using this API requires that your EC2 Instance has an IAM role with `cloudformation:SignalResource`
permissions. The [asg-rolling-deploy-dynamic module](/modules/asg-rolling-deploy-dynamic) creates these permissions for
you automatically.