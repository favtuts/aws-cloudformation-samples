# WP SureStack

With WP SureStack, you can create a WordPress stack that is performant, scalable, cost-effective, and easy to maintain.

WP SureStack uses a combination of plugins and services to help optimize your WordPress setup on AWS.  
We also packed in a lot of useful features into our CloudFormation template and pre-configured server, so you can hit the ground running with a working WordPress site in minutes.

If you’re thinking about creating your next WordPress site but don’t have the time to ramp up on learning AWS and Linux, give WP SureStack a try.

## Features

WP SureStack uses the following technologies:

* **Nginx**: a fast and light-weight web server
* **Nginx FastCGI Cache**: provides page-level caching at the web server level
* **Cache Sniper for Nginx**: WordPress plugin providing cache invalidation
* **Amazon Linux**: CentOS-based operating system, optimized for AWS
* **S3**: durable storage for user-generated image assets
* **CloudFront**: provides fast and efficient object caching
* **Aurora**: managed database layer with high-availability features
* **W3 Total Cache**: WordPress plugin that integrates your WordPress site with S3 and CloudFront
* **CloudFormation**: scripts and automates your WordPress infrastructure

## Installation and Usage

For setup instructions, click [here](https://bitbucket.org/thorntechnologies/wpsurestack-public/wiki/Setup).

## Links

* [Product Website](https://www.thorntech.com/products/wpsurestack/)
* [EULA](https://s3.amazonaws.com/thorntech-public-documents/eula/WPSureStackEULA.txt)
* [Issue Tracking](https://bitbucket.org/thorntechnologies/wpsurestack-public/issues)
* [CloudFormation Template](https://s3.amazonaws.com/thorntech-public-documents/cf-templates/WPSureStack.json) - No setup is necessary if you use this CloudFormation template. This template uses the proper AMI based on your region.
* [List of all wiki pages](https://bitbucket.org/thorntechnologies/wpsurestack-public/wiki/browse/)

## Support

Email support is available to Amazon Web Services Marketplace Customers at [support@thorntech.com](mailto:support@thorntech.com). We do not offer refunds, but you may terminate your AMI or CloudFormation Stack at any time.

## Troubleshooting

For troubleshooting and frequently asked questions, refer to our [Troubleshooting Wiki Page](https://bitbucket.org/thorntechnologies/wpsurestack-public/wiki/Troubleshooting)
