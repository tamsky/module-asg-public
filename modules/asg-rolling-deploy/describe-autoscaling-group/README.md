**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-asg>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-asg/blob/master/modules/asg-rolling-deploy/describe-autoscaling-group/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Describe Auto Scaling Group

This folder contains Python scripts for querying information about an Auto Scaling Group (ASG):

* `get-desired-capacity.py`: This script is meant to be called from a Terraform 
  [external data source](https://www.terraform.io/docs/providers/external/data_source.html) to fetch the latest 
  desired capacity value for an ASG. This is used to ensure that if an ASG size is modified externally (e.g. by auto 
  scaling policies), that desired capacity is preserved when we deploy an update to the ASG with this module, rather 
  than reverting it to the hard-coded value in the Terraform configuration.

Note that if Terraform ever adds its own `aws_autoscaling_group` data source, the need for these scripts will go away. 




## Working with the Python code

This section describes background info for maintainers of this module.  


### Why Python? And what's with the boto3 zip file?

Terraform's external data source runs on the local computer, so any code we run in it should be fairly portable. Our
typical solution for portability is to use standalone Go binaries, but that requires packaging a separate binary for
every OS. That means we'd still need some *other* language to pick the right binary *and* we'd have to either shove
all those binaries into this module (which would take a while to download) or add still more code to download the
proper binary at runtime.

Instead, for now, we are trying to use a language that's available on *most* operating systems: Python. Moreover, we've
included our sole dependency, the [boto3 AWS SDK](https://github.com/boto/boto3), as a single zip file. This solution
is hopefully portable *enough*.


### How do you run the code locally?

```
python get-desired-capacity.py --aws-region us-east-1 --tag-key foo --tag-value bar --default 3
```


### How do you update the boto version?

To update to version `VERSION` of boto:

1. Remove the previous version:

    ```
    rm -rf boto3-1.4.4.zip
    ```

1. Use `pip` to install the new version of boto, along with all of its dependencies, into a local folder:

    ```
    mkdir boto3-<VERSION>
    cd boto3-<VERSION>
    pip install -t . boto3
    ```

1. Compress the contents of the folder into a zip file and delete the folder:

    ```
    zip -r ../boto3-<VERSION>.zip .
    cd ..
    rm -rf boto3-<VERSION>
    ```

1. Update the version of boto3 at the top of `get-desired-capacity.py`.

