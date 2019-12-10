---
title:  "One thing to do if AWS accounts are changed on your Terraform project"
excerpt: "Getting puzzled by permissions issues when you change AWS accounts on an existing terraform project?  See how to fix this issue quickly!"
header:
 image: https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2019/12/blue+lake-hd.jpg
 caption: "Lake Tikitapu (near Rotorua), New Zealand"
 teaser: https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2019/12/blue+lake-tn.jpg
categories: 
  - Tech
tags:
  - terraform
---
This issue took a while for me to figure out, so hopefully, this will help someone else.

As we know, [Terraform](https://www.terraform.io/) can be used to create/update/destroy resources on all the different cloud providers *(AWS, Azure, Google etc)*.

One of the first things you do with your project is run `terraform init`. This will download the provider libraries and prepare your modules among other things. Next, you would run `terraform plan` to see the details about what would be changed.

If you change your AWS account used (say by updating the `$AWS_PROFILE` environment variable or editing the `~/.aws/credentials` file), and then run `terraform plan` again, you are likely to see an error like this:

```
Error: Error inspecting states in the "s3" backend:
 AccessDenied: Access Denied
 status code: 403, request id: AEC7B1948B85B1B5, host id: 6DXQ8ySWzcPQINptVOQLg/WmuM+644f3Rtt5AKUZ+iQRS8F/jGd1XGYek6c=
```

Errors like this are even more confusing when you can see in the logs that it is able to create the S3 bucket for the state and even the dynamodb used for the state locking. Why can it create some parts but not others?

Testing out access using the AWS CLI shows it is all working as well. So why would we be getting this error?

**If you change your AWS account you are deploying your terraform into after you have run `terraform init`, then you must *delete* your `./.terraform` folder and start again. It seems to cache your credentials and this needs to be refreshed if the AWS account is updated.**

Hopefully this will save others some time to figure this out!
