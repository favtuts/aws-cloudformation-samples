# Expanding Storage on your EC2 instance

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