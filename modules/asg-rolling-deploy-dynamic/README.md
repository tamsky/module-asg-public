**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-asg>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-asg/blob/master/modules/asg-rolling-deploy-dynamic/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Dynamic Auto Scaling Group with Rolling Deployment Module

This Terraform Module creates an Auto Scaling Group (ASG) that can do a zero-downtime rolling deployment. That means
every time you update your app (e.g. publish a new AMI), all you have to do is apply your Terraform templates and
the new version of your app will automatically roll out across your Auto Scaling Group. Note that this module *only*
creates the ASG and it's up to you to create all the other related resources, such as the launch configuration, ELB,
and security groups.

This module allows you to size the ASG both statically (e.g. by manually setting a fixed size) or dynamically (e.g.
via Auto Scaling Policies). To accomplish this, we use [CloudFormation](https://aws.amazon.com/cloudformation/) under
the hood. For an alternative that is purely-Terraform but only supports statically-sized ASGs, see the
[asg-rolling-deploy-static module](/modules/asg-rolling-deploy-static).

## What's an Auto Scaling Group?

An [Auto Scaling Group](https://aws.amazon.com/autoscaling/) (ASG) is used to manage a cluster of EC2 Instances. It
can enforce pre-defined rules about how many instances to run in the cluster, scale the number of instances up or
down depending on traffic, and automatically restart instances if they go down.

## How does rolling deployment work?

Since Terraform does not have rolling deployment built in (see https://github.com/hashicorp/terraform/issues/1552), we
are creating the Auto Scaling Group using [CloudFormation](https://aws.amazon.com/cloudformation/) and leveraging
CloudFormation's built-in [rolling deployment
UpdatePolicy](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-attribute-updatepolicy.html). As a
result, every time you update your launch configuration (e.g. by specifying a new AMI to deploy), you get:

1. An automatic rolling deployment of the new code across your Auto Scaling Group. To deploy a v2.0 of of your app,
   CloudFormation will deploy new EC2 Instances that run v2.0, wait for them to signal they are up and running (more
   on this in the next section), and then remove the EC2 Instances running v1.0.
1. The ability to monitor the deployment progress in the [CloudFormation
   Console](https://console.aws.amazon.com/cloudformation/home).
1. Automatic rollback to the previous version whenever there is an error deploying the new version.

## How do you use this module?

Check out the [asg-rolling-deploy-dynamic examples](/examples/asg-rolling-deploy-dynamic).

See the following sections for parameters this module supports.

#### Signaling to CloudFormation that your app is running

CloudFormation supports four flags that control how it determines if your app is running. The four flags and their
default values are:

```hcl
module "my_asg" {
  source = "git::git@github.com:gruntwork-io/module-asg.git//modules/asg-rolling-deploy-dynamic?ref=v1.0.8"

  wait_on_resource_signals = true
  rolling_deploy_pause_time = "PT5M"
  rolling_deploy_min_successful_instances_percent = 100
  creation_signal_count = 1
}
```

If you set `wait_on_resource_signals` to `true`, then after deploying each update to your ASG, CloudFormation will wait
up to `rolling_deploy_pause_time` (which uses ISO8601 duration format, so the default value of `PT5M` means 5 minutes)
for `rolling_deploy_min_successful_instances_percent` percent of your instances (by default, 100%) to explicitly send a
signal to it that they have started successfully. Note that on the very first deployment of your ASG, instead of
waiting for `rolling_deploy_min_successful_instances_percent`, CloudFormation just waits for a fixed number of signals
determined by the `creation_signal_count` variable.

There are three ways to handle these signals:

1. [cfn-signal python script](#cfn-signal-python-script)
1. [cfn-signal go binary](#cfn-signal-go-binary)
1. [Disable cfn-signal](#disable-cfn-signal)

#### cfn-signal Python script

AWS provides a Python script called
[cfn-signal](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-signal.html) that you can use to signal
CloudFormation.

* To **install** the `cfn-signal` script, you can use the [cloudformation-scripts
  module](/modules/cloudformation-scripts).
* To **run** the `cfn-signal` script, define a shell script in [User
  Data](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html#user-data-shell-scripts) and run `cfn-signal`
  when your app is ready to serve requests.

  User Data scripts are executed as the root user, so you need to use the absolute path for `cfn-signal`. On Amazon
  Linux, it will be:

  ```bash
  #!/bin/bash
  /opt/aws/bin/cfn-signal --success true --resource MyAsg --stack MyAsg
  ```

  On other flavors of Linux, as installed by the [cloudformation-scripts module](/modules/cloudformation-scripts), it
  will be:

  ```bash
  #!/bin/bash
  /usr/local/bin/cfn-signal --success true --resource MyAsg --stack MyAsg
  ```

  Note that the name of the resource (the ASG in this case) and the CloudFormation stack is going to be the same: it's
  the value of the `name` variable you pass into the asg-rolling-deploy-dynamic module. See the
  [cfn-signal documentation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-signal.html) for
  more info on the script.

#### cfn-signal Go binary

For some deployments, installing Python, or the particular version of Pyton (2.x) used by the cfn-signal is not an
option, so we have created a cross-platform Go binary as an alternative. See the [cfn-signal-go
module](/modules/cfn-signal-go) for details.

#### Disable cfn-signal

If you don't want to use `cfn-signal`, you should set `wait_on_resource_signals` to `false` and omit the
`rolling_deploy_min_successful_instances_percent` parameter. In this case, CloudFormation will wait up to
`rolling_deploy_pause_time` for the EC2 Instances to be marked as "Healthy." This is easier to set up, but less robust,
as "Healthy" only means the EC2 Instance is reachable, but not necessarily that your software is running correctly on
it.

#### Auto Scaling Policies

If you are **manually** controlling the size of this ASG, set the `desired_size` parameter to a fixed value:

```hcl
module "my_asg" {
  source = "git::git@github.com:gruntwork-io/module-asg.git//modules/asg-rolling-deploy-dynamic?ref=v1.0.8"

  min_size = 2
  max_size = 10
  desired_size = 5
}
```

If you are **automatically** controlling the size of this ASG using auto scaling policies, then you should **omit** the
`desired_size` parameter. For example, to automatically increase the number of instances every day at 9am and decrease
the number of instances at 5pm, you could do the following:

```hcl
module "my_asg" {
  source = "git::git@github.com:gruntwork-io/module-asg.git//modules/asg-rolling-deploy-dynamic?ref=v1.0.8"

  min_size = 2
  max_size = 10
}

resource "aws_autoscaling_schedule" "add_instances_in_the_morning" {
  scheduled_action_name = "add-instances-in-the-morning"
  desired_capacity = 10
  recurrence = "0 9 * * *"
  autoscaling_group_name = "${module.my_asg.asg_name}"
}

resource "aws_autoscaling_schedule" "remove_instances_in_the_evening" {
  scheduled_action_name = "remove-instances-in-the-evening"
  desired_capacity = 2
  recurrence = "0 17 * * *"
  autoscaling_group_name = "${module.my_asg.asg_name}"
}
```



