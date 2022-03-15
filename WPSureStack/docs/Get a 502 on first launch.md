Whenever you get a `502` error, the first place to look is the error log.

```
cat /var/log/nginx/wordpress-error.log 
```

Occasionally, you might see this error:

```
FastCGI sent in stderr: "PHP message: WordPress database error Table 'wpsurestack.wp_w3tc_cdn_queue' doesn't exist
```

This error is caused by a certain table not getting created by W3TC when WordPress gets wired up to CloudFront. The CloudFormation script is supposed to fix this, but there's a chance that the fix itself errors out.

If you are seeing `wp_w3tc_cdn_queue` in the logs, the first thing to try is to re-run the fix:

```
sudo su
cd /var/www/html/wordpress
wp db query < /home/ec2-user/w3tc_cdn.sql --allow-root
```

This takes you to your docroot. Then, using `wp db query`, it uses your wp-config credentials to run the sql.