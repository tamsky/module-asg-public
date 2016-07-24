**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-asg>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-asg/blob/master/examples/asg-rolling-deploy-dynamic/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Dynamic Auto Scaling Group with Rolling Deploy Examples

This folder shows examples of how to use the [asg-rolling-deploy-dynamic module](/modules/asg-rolling-deploy-dynamic)
to create an Auto Scaling Group (ASG) that supports rolling deployment:

* [with-elb](./with-elb): An example of how to deploy an ASG where each instance registers with an Elastic Load
  Balancer (ELB).
* [without-elb](./without-elb): An example of how to deploy an ASG without an ELB.
* [without-cfn-signal](./without-cfn-signal): An example of how to deploy an ASG without using `cfn-signal` to signal
  CloudFormation.

On each EC2 Instance in the ASG, for demonstration and testing purposes, we run a dummy web server that just returns
"Hello World". If you update this app (e.g. change the text to "Hello, World v2" or replace the app with a totally
different AMI of your own), the next time you run `terraform apply`, the new version of the app will roll out
automatically, with no downtime, across your ASG.

The asg-rolling-deploy-dynamic module allows you to size the ASG both statically (e.g. by manually setting a fixed
size) or dynamically (e.g. via Auto Scaling Policies). For an alternative that is purely-Terraform but only supports
statically-sized ASGs, see the [asg-rolling-deploy-static example](/examples/asg-rolling-deploy-static).

## How do you run these examples?

To run these examples, you need to do the following:

1. Build an AMI
1. Apply the Terraform templates

Each of these steps is described next.

#### Build an AMI

The `with-elb` and `without-elb` examples run a custom [Amazon Machine Image
(AMI)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) on each EC2 Instance in the ASG. The
contents of the AMI are defined in a [Packer](https://www.packer.io/) template under `packer/build.json`. To build an
AMI from this template:

1. Install [Packer](https://www.packer.io/).
1. Set up your [AWS credentials as environment variables](https://www.packer.io/docs/builders/amazon.html).
1. Run `packer build build.json` to create new AMIs in your AWS account. Note down the ID of this new AMI.

#### Apply the Terraform templates

To apply the Terraform templates:

1. Install [Terraform](https://www.terraform.io/), minimum version: `0.6.11`.
1. Open `vars.tf`, set the environment variables specified at the top of the file, and fill in any other variables that
   don't have a default. This includes setting the `ami` variable to the ID of the AMI you just built.
1. Run `terraform get`.
1. Run `terraform plan`.
1. If the plan looks good, run `terraform apply`.

## Deploying a new version

Once you have the app up and running, to test out the rolling deployment, make a change to any of the variables passed
into the ASG. For example, as a quick test, you could change the `server_text` variable to "Hello, World, v2.0!". Once
you've made your changes, do the following:

1. Commit your changes to Git so your teammates have access to them.
1. Run `terraform plan`.
1. If the plan looks good, run `terraform apply`.

The new version of the app will automatically deploy across the Auto Scaling Cluster. Under the hood, the rolling
deployment is done by [CloudFormation](https://aws.amazon.com/cloudformation/), so you can monitor the progress of the
deployment in the [CloudFormation Console](https://console.aws.amazon.com/cloudformation/home). If anything goes wrong
during deployment, the new version will automatically be rolled back and the old version will continue running.
