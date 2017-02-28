**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-asg>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-asg/blob/master/examples/asg-rolling-deploy/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Auto Scaling Group with Rolling Deploy Examples

This folder shows examples of how to use the [asg-rolling-deploy module](/modules/asg-rolling-deploy)
to create an Auto Scaling Group (ASG) that supports rolling deployment:

* [with-elb](./with-elb): An example of how to deploy an ASG where each instance registers with an Elastic Load 
  Balancer (ELB).
* [without-elb](./without-elb): An example of how to deploy an ASG without any load balancer.

On each EC2 Instance in the ASG, for demonstration and testing purposes, we run a dummy web server that just returns
"Hello World". If you update this app (e.g. change the text to "Hello, World v2" or replace the app with a totally
different AMI of your own), the next time you run `terraform apply`, the new version of the app will roll out
automatically, with no downtime, across your ASG.




## How do you run these examples?

1. Install [Terraform](https://www.terraform.io/).
1. Make sure you have Python installed (version 2.x) and in your `PATH`.
1. Open `vars.tf`, set the environment variables specified at the top of the file, and fill in any other variables that
   don't have a default.
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

The new version of the app will automatically deploy across the Auto Scaling Cluster. Under the hood, this is done by
taking advantage of Terraform's `create_before_destroy` lifecycle property. This is the same strategy used by HashiCorp
for it's own rolling deployments, as [as described by Paul Hinze
here](https://groups.google.com/forum/#!msg/terraform-tool/7Gdhv1OAc80/iNQ93riiLwAJ).
