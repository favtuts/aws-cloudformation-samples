# WP CloudStack - WordPress optimized for AWS
Production-ready, professional-grade WordPress stack built for performance and scalability. Uses Nginx FastCGI cache on standard Amazon Linux, and integrates with S3, CloudFront, and Aurora.

# Product Overview

WP CloudStack is a production-ready, professional-grade WordPress stack built for performance and scalability.
With WP CloudStack, you don't need to be a cloud expert or Linux administrator to leverage the power of AWS for your WordPress site.
WP CloudStack runs Nginx with FastCGI cache on standard Amazon Linux for lightweight performance, ease-of-use, and minimum maintenance.
It is pre-configured with industry-standard best practices for security, reliability, and performance, including:
* Image asset off-loading to durable storage on S3
* File permission security
* Pre-configured network infrastructure
* Database backups with Amazon Aurora
* Gzip compression
* Browser caching support
* Content delivery using AWS CloudFront

# Highlights
* Runs Nginx with FastCGI cache on standard Amazon Linux for lightweight performance, ease-of-use, and minimum maintenance.
* Automated integration with S3, CloudFront (CDN), and AuroraDB via CloudFormation template.
* Additional optimizations that improve caching, security, and memory monitoring

# Deployment Architecture

![WPCloudStack-Architecture](WPCloudStack-Architecture.png)

# CloudFormation Template

* [WP CloudStack CloudFormation JSON Template](WPCloudStack-CF-Template.json)
* [WP CloudStack CloudFormation YAML Template](WPCloudStack-CF-Template.yml)

# Usage Instructions
Setup using the CloudFormation template is recommended. Detailed instructions can be found on our [WP CloudStack Setup](docs/) 
