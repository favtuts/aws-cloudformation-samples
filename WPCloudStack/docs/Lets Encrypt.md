Let's Encrypt is a popular tool for getting free SSL certificates.

Here's a userful article on how to set it up on Amazon Linux:
https://medium.com/@gnowland/deploying-lets-encrypt-on-an-amazon-linux-ami-ec2-instance-f8e2e8f4fc1f

This is a sort of checklist to follow along for Amazon Linux (of which WP SureStack is based on).

Here are a few needed pieces of information, which I used for the sake of example:

* **Domain**: robtest.thorn.tech
* **IP address**: 54.210.100.47
* **Email**: robert.chen@thorntech.com

(a) Create a Host A record in Route 53, pointing `robtest.thorn.tech` to `54.210.100.47`. Wait 10-15 minutes to give DNS a chance to propagate. Otherwise, you get an error on a later validation step.

(b) Sudo to root

`sudo su`

(c) Install dependencies, but these may already be installed

`yum install python27-devel git`

(d) Download letsencrypt to `/opt/letsencrypt`

`git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt`

(e) Run the letsencrypt wizard. The article mentioned above says it's important to include the debug flag.

`/opt/letsencrypt/letsencrypt-auto --debug`

(f) Choose 2, for Nginx. Let's encrypt will do things to your nginx conf.

```
How would you like to authenticate and install certificates?
-------------------------------------------------------------------------------
1: Apache Web Server plugin - Beta (apache)
2: Nginx Web Server plugin - Alpha (nginx)
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
```

(g) Supply your email address (for renewal notification purposes)

```
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): robert.chen@thorntech.com
```

(h) Accept the terms

```
-------------------------------------------------------------------------------
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v01.api.letsencrypt.org/directory
-------------------------------------------------------------------------------
(A)gree/(C)ancel: A
```

(i) You don't need to share your email 

```
-------------------------------------------------------------------------------
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about EFF and
our work to encrypt the web, protect its users and defend digital rights.
-------------------------------------------------------------------------------
(Y)es/(N)o: N
```

(j) Make a note of which nginx conf file that letsencrypt is making changes to. In my case, I was using a test box (didn't do this on WP CloudStack, so it's using virtual.conf)

```
Deploying Certificate to VirtualHost /etc/nginx/conf.d/virtual.conf
```

(k) Choose **2** to allow letsencrypt to generate SSL redirect syntax for you. 

```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
```

(L) Check your nginx conf file for any changes. Letsencrypt might add additional server blocks. Feel free to clean up and consolidate. But note that they now point your ssl certificate and key locations to the letsencrypt certificates:

```
    ssl_certificate /etc/letsencrypt/live/robtest.thorn.tech/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/robtest.thorn.tech/privkey.pem; # managed by Certbot
```

(m) Restart nginx

```
nginx -t
service nginx restart
```