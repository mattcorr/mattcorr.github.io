---
title:  "How to define output values for dynamically created terraform resources"
excerpt: "Not sure how to get output details with for_each loop created terraform resources? Use the zipmap function."
header:
 image: https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2020/08/dove-lake-hd.jpg
 caption: "Dove Lake, Cradle Mountain, Tasmania"
 teaser: https://blog-ii-images.s3-ap-southeast-2.amazonaws.com/2020/08/dove-lake-tn.jpg
categories: 
  - Tech
tags:
  - terraform
---
Looking at the standard [documentation page for terraform output](https://www.terraform.io/docs/configuration/outputs.html) there are some samples for basic values and for how to access module values.

This is great, but what if you had been following some of [my previous posts](https://www.intrepidintegration.com/tech/how-to-reference-data-objects-with-terraform/) about looping and want get some output for resources that created with the `for_each` command? How can the static structure of the examples shown work for a dynamically created list of resources?

The standard syntax for an output statement is:

```hcl
output "sqs_output" {
  value = aws_sqs_queue.message_queue.arn
}
```
**NOTE:** Displaying the ARN is not particular useful, but its good for demo purposes.
{: .notice--warning}

This will give us `terraform apply` output of something like:
```
Outputs:

sqs_output = arn:aws:sqs:ap-southeast-2:658977328088:simple-demo-sqs.fifo
```

But if we had created a dynamic number of sqs queues, how should we structure the output block?

One way I prefer, is to utilise the existing [zipmap function](https://www.terraform.io/docs/configuration/functions/zipmap.html) with the following syntax: 
```hcl
output "sqs_output" {
  value = zipmap( values(aws_sqs_queue.message_queue)[*].name, values(aws_sqs_queue.message_queue)[*].arn ) 
}
```

This will give us `terraform apply` output of:
```
Outputs:

sqs_output = {
  "matt_test_one.fifo" = "arn:aws:sqs:ap-southeast-2:658977328088:matt_test_one.fifo"
  "matt_test_three.fifo" = "arn:aws:sqs:ap-southeast-2:658977328088:matt_test_three.fifo"
  "matt_test_two.fifo" = "arn:aws:sqs:ap-southeast-2:658977328088:matt_test_two.fifo"
}
```
Nice and simple.

Feel free to try this yourself in my [sample terraform project](https://github.com/mattcorr/tf-presentation-demo) in the [looping](https://github.com/mattcorr/tf-presentation-demo/tree/master/looping) folder.

This a simple solution to get what we are after! But if there is a better way you have seen, feel free to let me know via the comments below.