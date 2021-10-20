---
title:  "How to reference data objects via for_each with Terraform"
excerpt: "See how you can easily reference more complex data types with the for_each command in Terraform."
header:
 image: https://blogresourcestorage.blob.core.windows.net/images/2019/12/wairoatown-hd.jpg
 caption: "Wairoa, New Zealand"
 teaser: https://blogresourcestorage.blob.core.windows.net/images/2019/12/wairoatown-tb.jpg
categories: 
  - Tech
tags:
  - terraform
---

I have been skilling up on [Terraform](https://www.terraform.io/) over the last few weeks and have been enjoying it. One of my tasks was to upgrade an existing project from Terraform 0.11 to 0.12. One of the new features in 0.12.6 and later was the introduction of the `for_each` function.

How has that been helpful?

For the examples in this blog post, for simplicities sake, we are using [SQS](https://aws.amazon.com/sqs/) resources in [AWS](https://aws.amazon.com). There is nothing stopping you from use [Azure](https://azure.microsoft.com/en-us/) or [GCP](https://cloud.google.com/).
{: .notice--info}

In the past, if you wanted to define a large number of similar resources in Terraform you could pass a list to the resource.

The variable is defined as:
```hcl
variable "sqs_names" {
    type  = list(string)
}
```

The variable is populated as:
```hcl
sqs_names = [
    "matt_test_one", 
    "matt_test_two", 
    "matt_test_three"]
```

The resource (prior to terraform 0.12.6) is defined as:
```hcl
resource "aws_sqs_queue" "message_queue" {
  count          = length(var.sqs_names)

  name           = format("%s.fifo", element(var.sqs_names, count.index))
  fifo_queue     = true
}
```

Having this configuration will create three SQS resources when `terraform apply` is run.
```
aws_sqs_queue.message_queue[2]: Creating...
aws_sqs_queue.message_queue[0]: Creating...
aws_sqs_queue.message_queue[1]: Creating...
aws_sqs_queue.message_queue[2]: Creation complete after 0s [id=https://sqs.ap-southeast-2.amazonaws.com/3152243/matt_test_three.fifo]
aws_sqs_queue.message_queue[0]: Creation complete after 0s [id=https://sqs.ap-southeast-2.amazonaws.com/3152243/matt_test_one.fifo]
aws_sqs_queue.message_queue[1]: Creation complete after 0s [id=https://sqs.ap-southeast-2.amazonaws.com/3152243/matt_test_two.fifo]

Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```

This is pretty neat right? But the issue is, if you change your list variable to include some more queue names:
```hcl
sqs_names = [
    "matt_test_one", 
    "matt_test_one_again",
    "matt_test_two", 
    "matt_test_three"]
```

Then the next time `terraform plan` is run, you will see as the summary:
```
Plan: 3 to add, 0 to change, 2 to destroy.
```

What? We didn't change the other queues? Why do they need to be recreated again? 

This is due to the resource being tied to the list index. This can be confirmed by checking out the state file with `terraform state list`

```
aws_sqs_queue.message_queue[0]
aws_sqs_queue.message_queue[1]
aws_sqs_queue.message_queue[2]
```

So if we change the list, potentially more than one resource will be recreated. One workaround is to only append to the end of the list, but that feels really brittle and not a proper solution. So what can we do?

Enter the `for_each` command!

We can still keep the list, but tweaking our resource code to be something like this:

```hcl
resource "aws_sqs_queue" "message_queue" {
  for_each = toset(var.sqs_names)

  name           = format("%s.fifo", each.key)
  fifo_queue     = true
}
```

We no longer need to be careful of the list variable order, we can insert/delete/update elements as needed and only the impacted resources are recreated. This can be confirmed by looking at the state after it is successfully applied.
```
aws_sqs_queue.message_queue["matt_test_one"]
aws_sqs_queue.message_queue["matt_test_one_again"]
aws_sqs_queue.message_queue["matt_test_three"]
aws_sqs_queue.message_queue["matt_test_two"]
```

This is a great improvement, but what if we want to have a more complicated object rather than a simple list? This can be useful to set multiple properties of a resource rather than just the name as we have been doing so far.

Instead of having just a list variable defined as:

```hcl
variable "sqs_names" {
    type  = list(string)
}
```
we could use a **map of objects** like this:

```hcl
variable "sqs_data" {
  type = map(object({
    delay        = number
    max_msg_size = number
    environment  = string
  }))
}
```
An example of the variable definition would be:
```hcl
sqs_data = {
    matt_test_one = {
        delay = 10
        max_msg_size = 1024
        environment = "dev"
    },
    matt_test_two = {
        delay = 10
        max_msg_size = 2048
        environment = "test"
    },
    matt_test_three = {
        delay = 5
        max_msg_size = 4096
        environment = "prod"
    }
}
```

The allows us to define what ever we want to be variable for any resources that need to be created/updated in bulk.

The updated resource definition would be:

```hcl
resource "aws_sqs_queue" "message_queue" {
  for_each = var.sqs_data

  delay_seconds    = each.value["delay"]
  max_message_size = each.value["max_msg_size"]
  name             = format("%s.fifo", each.key)
  fifo_queue       = true
  tags = {
    Environment = each.value["environment"]
  }
}
```
When using a list of strings, the `each.key` and `each.value` fields are the same thing. But when we pass in a map of objects, the `each.key` refers to the name, and the `each.value` is an array of the values which can be accessed as shown above.



The `terraform plan` will have the additional fields set for each object in the map
```
  # aws_sqs_queue.message_queue["matt_test_three"] will be created
  + resource "aws_sqs_queue" "message_queue" {
      + arn                               = (known after apply)
      + content_based_deduplication       = false
      + delay_seconds                     = 5
      + fifo_queue                        = true
      + id                                = (known after apply)
      + kms_data_key_reuse_period_seconds = (known after apply)
      + max_message_size                  = 4096
      + message_retention_seconds         = 345600
      + name                              = "matt_test_three.fifo"
      + policy                            = (known after apply)
      + receive_wait_time_seconds         = 0
      + tags                              = {
          + "Environment" = "prod"
        }
      + visibility_timeout_seconds        = 30
    }
  ```

  I have also found the resource terraform created when using `for_each` for either lists or maps of object is more readable and maintainable than using the count.index approach. Using the maps of objects approach is more ideal when there are large groups of similar objects that need to be created.

  So there you have it! Some examples of how to use `for_each` in action!   
  If you want to play with these samples in a complete terraform project, refer to this [github project here](https://github.com/mattcorr/terraform-demo).
  
See you next time!

