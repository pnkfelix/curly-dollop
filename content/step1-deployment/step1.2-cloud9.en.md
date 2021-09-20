---
title : "Setting up a Cloud9 environment"
weight : 22
---

Much of today's material is applicable to any development environment\: personal
machine, or cloud desktop.

However, rather than write out instructions for every possible setting, these
instructions are going to assume you are using a Cloud9 environment. Mapping the
instructions provided here to other contexts is left as an exercise for the
reader.

## Initial Setup of Cloud9

To setup a Cloud9 desktop, go to the AWS Console and choose the "Cloud9" service;
it is listed with "Developer Tools" under the "All Services" list.

Once there, hit the "Create environment" button, and fill out the form.

On the next step\:

 * For environment type, I use "Create a new EC2 instance for
environment (direct access)"
 * For instance type, I use `m5.large`. (A smaller instance type might suffice; I have not had a chance to experiment with running these steps with a smaller instance type.)
 * For platform I use "Amazon Linux 2".
 * For the cost-saving setting, the default of hibernating after 30 minutes is a reasonable choice for this workshop.

Hit "next step" again, review the settings, and confirm them.

Now wait for the Cloud9 environment to be created.

## Increasing storage capacity

Once the environment is created, you will need to increase the volume size of
its associated storage to something more reasonable; 10 Gigabytes isn't going to
cut it for what we are doing, unfortunately.

And, also unfortunately, I do not know a way to do this trivially within the
Cloud9 environment itself.

But, there is a helper script provided by the Cloud9 team, here, under the section
entitled "Resize an Amazon EBS volume used by an environment"\:

https://docs.aws.amazon.com/cloud9/latest/user-guide/move-environment.html

Follow the instructions there to create a `resize.sh` script, and use that to
resize the drive to 50 GB.

```sh
sh resize.sh 50
```

You can double-check on how much size you have by running `df`\:

```sh
df -kh
```

## Logging back into Cloud9

If you get logged out from your Cloud9 environment (or from your AWS account),
you can reaccess it by going back to the AWS Cloud9 site and looking for it
under "Your environments"; the "Open IDE" button there will re-open the
web-based IDE as you left it.
