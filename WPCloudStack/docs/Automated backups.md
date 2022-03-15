There are a number of plugins to automatically back up your WordPress files and database.

If you wish to do this via CLI, here's how:

First, create an S3 bucket to store your backups. (It's important that this bucket is not the one you're using to serve your web content.) For example, create a bucket named `rob-test-surestack-backup` (make sure to change this value) throughout the rest of the article.

Second, grant your EC2 instance access to this bucket. 

* In the AWS console, select your EC2 instance
* In the EC2 details, click on **IAM role**
* On the right, click **Add inline policy**
* Select the **JSON** tab 
* Paste in the following (remember to change the S3 bucket name): 

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:PutObject",
            "Resource": [
                "arn:aws:s3:::rob-test-surestack-backup/*"
            ]
        }
    ]
}
``` 

This gives your EC2 instance the permission to write to the backup S3 bucket. The key here is that you are restricting the permission to `s3:PutObject`, so server does not have the ability to delete nor snoop around your backup S3 bucket.

Third, create the following script:

```
/home/ec2-user/backup.sh
```

Make it executable:

```
chmod +x /home/ec2-user/backup.sh
```

And paste in the following contents:

```
#!/bin/bash

if [[ "$1" == "" ]] || [[ `whoami` != "root" ]]; then
    echo "Usage: sudo $0 rob-test-surestack-backup"
    exit 1
fi

# usage: ./backup.sh rob-test-surestack-backup
BUCKET=$1
# takes the docroot as the second parameter, or defaults to /var/www/html/wordpress
DOCROOT=${2:-/var/www/html/wordpress}

#
# back up WordPress files
#

# back up the entire html folder, because some folks move the wp-config file to the parent folder
cd /var/www
# create a zipped tar archive named html-yyyy-mm-dd.tar.gz
tar czvpf html-$(date +%Y-%m-%d).tar.gz html
# use the AWS CLI to copy this file to the S3 bucket
aws s3 cp html-????-??-??.tar.gz s3://$BUCKET/
# clean up the file
rm -f html-????-??-??.tar.gz

#
# back up database
#

# do this in a directory that is outside of the docroot
cd /var/www
# use the WP CLI to export the database
# point the path to the docroot, since we're doing this from outside the docroot
# allow-root lets you run this command as root
# the hyphen (-) sends the database backup contents to standard-out
# the standard-out gets piped into gzip
# and the gzip contents are dumped into a file db-yyyy-mm-dd.gz
wp db export --path=$DOCROOT - --allow-root | gzip > db-$(date +%Y-%m-%d).gz
# copy the file to S3
aws s3 cp db-????-??-??.gz s3://$BUCKET/
# clean up the file
rm -f db-????-??-??.tar.gz
```

Run the script with this command:

```
cd /home/ec2-user/
sudo ./backup.sh rob-test-surestack-backup
```

To run this weekly, edit the root user's crontab:

```
sudo su
crontab -e
```

And add in the following entry:

```
* * */7 * * /home/ec2-user/backup.sh rob-test-surestack-backup >/dev/null 2>&1
```

(Again, remember to change the S3 bucket destination.)