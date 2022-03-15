# Troubleshooting

## How to Expand Storage on your EC2 instance

The best approach would be to resize the EC2 instance volume. AWS provides instructions on how to do so here:

http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-expand-volume.html

This document can be pretty intimidating. But here's a condensed version:

* First, you want to back up your instance. You can do this by creating a snapshot.
* Next, you can resize the EBS volume. Go to EC2 > Volumes and find your instance's volume.
* Go to Actions > Modify Volume, and increase the storage size. For now, specify 64 GB.
* It'll take a while for the volume resize operation to complete. But under your volume's details, look for the "state". Once it is "optimizing", you can proceed.
* Run the following (note the space between xvda and 1):
`sudo growpart /dev/xvda 1`

* Run the following: 
`sudo resize2fs /dev/xvda1`

Here's what's going on. Increasing the EC2 volume is a 3-step process. Imagine you're inflating 3 balloons nested inside each other. But you have to inflate the outer one first to make room for the inner ones.

1. The outermost layer is the EBS volume. You increased this via the "Modify Volume" step in the AWS console.
2. The middle layer is the partition. You increased this using `growpart`. You can use  `lsblk` to verify that you're using the full 64 GB volume.
3. The inner layer is the file system. You increased this using `resize2fs`. You use `df -h` to verify that you're using the full partition.


The documentation has a lot of explanation and tangents for edge cases that you can refer to. But the actual process is just Modify Volume in the AWS console, and two commands: `growpart` and `resize2fs`.



## Enabling Termination Protection on your CloudFormation Stack 

Termination protection is a great way to protect your stack from accidental deletion. This option is very easy to enable, whether you are creating a new CloudFormation stack or managing an existing stack.


### During creation:

At the "options" stage, click **Advanced** to expand the section. Click **Enable** next to "Termination protection".

### After creation:

At the CloudFormation console, click a stack name to view the stack details. On the upper right corner, find **Other Actions** and select **Change termination protection**. Click **Yes, Enable** .


## Default website does not come up

Sometimes, the default website does not come up at all. And when you check the `wp-config.php` file, it's completely blank and there is nothing in it (except for a line for `fs_method`).

A likely culprit is the password chosen for the database Master password. Certain characters such as `&` are perfectly valid for a MySQL password. However, these characters break the bash script as it tries to provision the database connection account.

To see which character is causing the issue, review the log located at `/var/log/cloud-init-output.log`. Search for `wpconnectionuser` and see if any portion of the password you provided appears. This should give a clue as to what character is breaking the script.

## A duplicate bucket name exists

S3 bucket names need to be unique, so it's possible that you or another organization is already using that S3 bucket name.

Another thing to keep in mind is that the S3 bucket remains when the CloudFormation stack is deleted. This is a design choice, just in case your production CloudFormation stack were accidentally deleted. It is up to you to manually delete the S3 bucket.