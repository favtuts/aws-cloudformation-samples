When visiting the site with `https://`, you get an error:

```
Your connection is not private
```

Using Chrome, it doesn't give you the ability to do: ADVANCED > Proceed (unsafe).

To fix this, edit the nginx conf file:

```
/etc/nginx/conf.d/wordpress.conf
```

And comment out the following line (pre-pend it with the `#` character) so it looks like this:

```
#add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
```

Then restart nginx:

```
service nginx restart
```

Clear the cache of your browser, since it would have cached the HSTS setting. On Chrome on Mac:

1. Open Developer Tools with `Command-Option-i`
1. Right-click on the refresh button (this only works if Developer Tools is open)
1. Select **Empty Cache and Hard Reload**

![empty-cache-reload.png](https://bitbucket.org/repo/8z7XepL/images/1286665317-empty-cache-reload.png)

In the browser, click **ADVANCED**, and click the link to:

```
Proceed to <ip address> (unsafe)
```