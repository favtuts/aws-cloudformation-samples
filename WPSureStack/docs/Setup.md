# WP SureStack setup using CloudFormation


WP SureStack relies on a handful of AWS services that help improve the performance and scalability of your WordPress site. Our CloudFormation template not only sets up your WordPress server, but it also integrates your WordPress site with S3, CloudFront, and AuroraDB.

CloudFormation lets you easily provision servers, databases, and other resources by describing them in a JSON document. CloudFormation is a great tool for provisioning your entire infrastructure in an automated fashion.  

Using our CloudFormation template, you can have a fresh WordPress instance running in a matter of minutes.

## Preliminary Setup

You need to create an EC2 Key Pair so that you can log into your EC2 instance.

Within the AWS console: 

1. Go to EC2 > Network & Security > Key Pairs
1. Click `Create Key Pair`
1. Enter a `Key pair name` and click `Create`
1. You will automatically download a `.pem` file. Store this in a safe place.

## CloudFormation Setup - Create a CloudFormation Stack

Download the following CloudFormation template: 

**[WPSureStack.json](https://s3.amazonaws.com/thorntech-public-documents/cf-templates/WPSureStack.json)**

1. Within the AWS console, go to **CloudFormation** and click **Create Stack**.
1. On the **Select Template** page, click the **Choose File** button to upload a template to Amazon S3.
1. Select the `WPSureStack.json` file you downloaded, then click **Next**.
1. On the **Create Stack** form, fill out the information as follows:

**Specify Details**

* **Stack name**: This is the name of your CloudFormation stack. For example, you can use `my-wp-site`.

**EC2 instance**

In this section, you configure settings specific to your EC2 instance. 

* **EC2Class**: Use the drop down to select an instance type. `t2.micro` is the cheapest and is great for experimentation. Use `m4.large` or better for heavier production loads.
* **InputCIDR**: This is the IP range that can SSH into the server. Enter your public IP address, followed by a `/32`. For example, `12.34.56.78/32`. You can use [checkip.dyndns.org](http://checkip.dyndns.org) to figure out your IP address. 
* **KeyName**: Select your Key Pair from the drop down list. If you can't find it, see the **Before you begin** section at the top of the page.

**Resource Naming**

This section is where you name some of your website resources.

* **BucketName**: Your image uploads will get copied to this S3 bucket, which acts as a CDN origin for CloudFront. Make sure the bucket name is unique.
* **WebsiteTitle**: This is the **Site Title** of your WordPress website.
* **WebsiteDomain**: You can leave this blank, and WP SureStack will just use your public IP as your URL. If you happen to have a domain name registered, enter it here. Just remember to point your DNS to the public IP address.

**Note**: You can change your domain name later on. Check our documentation to learn more.

**WordPress user account**

In this section, you configure a WordPress admin user which you can use to log into your website.

* **WPUser**: This is the username of the WordPress admin account.
* **WPPassword**: Remember the password because you'll need it once the stack is complete.
* **WPEmail**: This is the email address associated with the admin account. Please use a real email address, because this is also used for password reset.

**Aurora DB**

In this section, you configure settings for your Aurora DB instance.  
**You will log in with the username `admin`, with your chosen `DBPassword`**.

* **DBClusterName**: This is the name of your Aurora DB instance. Your Aurora instance can hold multiple databases, so name it something generic like "wpsurestack"
* **DBPassword**: This is the master password for the Aurora instance. Make sure this is secure. Don't worry -- you can reset this from the AWS console later on if you forget. 
* **DBClass**: Select a database instance type. `db.t2.small` is your cheapest option.

**Note**: The database connection string contains the account credentials for connecting to your WordPress database. This is generated automatically. You can find it in your `wp-config.php` file.

### Options page

On the Options page, click **Next**

### Review page

On the Review page, scroll to the bottom and check the box next to **I acknowledge that AWS CloudFormation might create IAM resources**.

Click **Create**

## Try it out

### Wait for the stack to complete

Wait until the CloudFormation template is done creating. While you're waiting, click on the **Events** tab to view its progress.

**Note**: This process takes roughly 25 minutes. Much of this time is spent on Aurora DB. 

When it's finished, click on the **Outputs** tab. Look for the IP address listed next to **EIPDNS**. This is what you will use as your website url, and for connecting via SSH.

### Visit the WordPress site

When the stack status reaches `CREATE_COMPLETE`, click on the **Outputs** tab. Copy and paste the value of the EIPDNS into your browser to view your new website.

Visit the `/wp-admin` URI and log in using the credentials you provided earlier. You should see the WordPress admin console.

### SSH into the server

To SSH into your EC2 instance:

1. `cd` to the location of your `.pem` key
1. Run `chmod 600 mykey.pem` to lock down your SSH key
1. Run `ssh ec2-user@<your ip address>`