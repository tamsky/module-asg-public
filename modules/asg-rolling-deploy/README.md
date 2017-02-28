**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-asg>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-asg/blob/master/modules/asg-rolling-deploy/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Auto Scaling Group with Rolling Deployment Module

This Terraform Module creates an Auto Scaling Group (ASG) that can do a zero-downtime rolling deployment. That means
every time you update your app (e.g. publish a new AMI), all you have to do is run `terraform apply` and the new 
version of your app will automatically roll out across your Auto Scaling Group. Note that this module *only*
creates the ASG and it's up to you to create all the other related resources, such as the launch configuration, ELB,
and security groups.




## What's an Auto Scaling Group?

An [Auto Scaling Group](https://aws.amazon.com/autoscaling/) (ASG) is used to manage a cluster of EC2 Instances. It
can enforce pre-defined rules about how many instances to run in the cluster, scale the number of instances up or
down depending on traffic, and automatically restart instances if they go down.




## How does rolling deployment work?

Since Terraform does not have rolling deployment built in (see https://github.com/hashicorp/terraform/issues/1552), we
are faking it using the `create_before_destroy` lifecycle property. This approach is based on the rolling deploy
strategy used by HashiCorp itself, [as described by Paul Hinze
here](https://groups.google.com/forum/#!msg/terraform-tool/7Gdhv1OAc80/iNQ93riiLwAJ). As a result, every time you
update your launch configuration (e.g. by specifying a new AMI to deploy), Terraform will:

1. Create a new ASG with the new launch configuration.
1. Wait for the new ASG to deploy successfully and for the instances to register with the load balancer (if you 
   associated an ELB or ALB with this ASG).
1. Destroy the old ASG.
1. Since the old ASG is only removed once the new ASG instances are registered with the ELB and serving traffic, there
   will be no downtime. Moreover, if anything went wrong while rolling out the new ASG, it will be marked as
   [tainted](https://www.terraform.io/docs/commands/taint.html) (i.e. marked for deletion next time) and the original
   ASG will be left unchanged, so again, there is no downtime.
      
Note that if all we did was use `create_before_destroy`, on each redeploy, our ASG would reset to its hard-coded
`desired_capacity`, losing the capacity changes from auto scaling policies. We solve this problem by using an 
[external data source](https://www.terraform.io/docs/providers/external/data_source.html) that runs the Python script 
[get-desired-capacity.py](describe-autoscaling-group/get-desired-capacity.py) to fetch the latest value of the 
`desired_capacity` parameter: 

- If the script finds a value from an already-existing ASG, we use it, to ensure that the changes form auto scaling 
  events are not lost. 
  
- If the script doesn't find an already-existing ASG, that means this is the first deploy, and we fall back to the 
  hard-coded `desired_capacity` value.




## How do you use this module?

Check out the [asg-rolling-deploy examples](/examples/asg-rolling-deploy).