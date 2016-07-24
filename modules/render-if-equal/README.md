**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-asg>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-asg/blob/master/modules/render-if-equal/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# Render If Equal Module

This module contains a hacky way to support if-statements in Terraform. Terraform does not have support for
[conditional logic](https://github.com/hashicorp/terraform/issues/1604), and usually you don't need it, but in the
rare cases where you do, this module contains a workaround.

## How do you use this module?

Let's say you want to render some text only if `var.x` is equal to `foo`. You would use this module as follows:

```hcl
module "if_x_equals_foo" {
  source = "git::git@github.com:gruntwork-io/module-asg.git//modules/render-if-equal?ref=v0.0.18"

  if_left = "${var.x}"
  equals_right = "foo"
  then = "this will render if left and right are equal"
  else = "this will render if left and right are not equal"
}
```

The `result` output of this module will now contain the the text you passed in as `then` if the values were equal or
the text you passed in as `else` otherwise. You can use this `result` output elsewhere:

```hcl
x = "${module.if_x_equals_foo.result}"
```

For an example usage, see the [asg-rolling-deploy-dynamic module](/modules/asg-rolling-deploy-dynamic).

## Caveats and warnings

1. This is obviously a hacky workaround, so only use if it you have no other choice!
2. The `if_left` parameter must be a plain string.
3. The `if_right` parameter is converted to a regular expression that matches the entire line:
   `"/^${var.equals_right}$/"`. This avoids accidental partial matches of `if_left`. This also means you can use regular
   expression syntax within `if_right`. For example, you could render something only `if_left` contains one or more
   digits by setting `if_right = "\\d+"`.
4. Do not set `else` to an empty string (`""`), or you'll get a Terraform `Diff's didn't match error`.