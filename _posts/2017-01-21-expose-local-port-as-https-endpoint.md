---
layout: post
title: Expose local port as HTTPS endpoint on the internet
---

I recently needed to expose local port of my dev machine to the internet. I realized I could combine SSH port forwarding, nginx and letsencrypt. Ubuntu 16.04 is my OS of choice on the server.

I did this mostly to try out letsencrypt. If this looks too complicated for you, you might want to consider using [ngrok](https://ngrok.com) or similar service.

## Port forwarding

Exposing local port using port forwarding is pretty simple assuming you have linux server which you can access by ssh. Let's expose our local port 8000 as port 80 on yourserver.example.com.

{% highlight bash %}
$ ssh -N user@yourserver.example.com -R 80:localhost:8000
{% endhighlight %}

This was easy. Now the harder part, HTTPS.

## Getting the certificate

I followed tutorial written by [Mitchell Anicas](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04). If you get lost on a way please refer back to it. First, let's install `nginx` and `letsencrypt`.

{% highlight bash %}
$ sudo apt-get update
$ sudo apt-get install nginx letsencrypt
{% endhighlight %}

Letsencrypt needs a way to verify that you control the domain name by visiting `http://yourserver.example.com/.well-known/SOMETHING`. We will use Webroot plugin to automatically create the file for us. Depending on your nginx configuration, you may need to explicitly allow access to the `/.well-known` directory.

Open default nginx site config:

{% highlight bash %}
$ sudo nano /etc/nginx/sites-available/default
{% endhighlight %}

Inside the `server` block, add new location:

{% highlight nginx %}
    location ~ /.well-known {
        allow all;
    }
{% endhighlight %}

Now apply our new configuration:

{% highlight bash %}
$ sudo systemctl reload nginx
{% endhighlight %}

Let's finally get the certificate!

{% highlight bash %}
$ sudo letsencrypt certonly -a webroot --webroot-path=/var/www/html -d yourserver.example.com
{% endhighlight %}

Letsencrypt will probably ask for your email and if everything succeeds you should be able to see your certificate in following directory(substituting in your domain name):

{% highlight bash %}
$ sudo ls -l /etc/letsencrypt/live/your_domain_name
{% endhighlight %}

## HTTPS proxy

HTTPS configuration is hard, luckily there are configurations we can just copy. It's a good practice to generate strong [Diffie-Hellman group](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange).

{% highlight bash %}
$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
{% endhighlight %}

Next, let's create snippet with decent SSL configuration.

{% highlight bash %}
$ sudo nano /etc/nginx/snippets/ssl-params.conf
{% endhighlight %}

{% highlight nginx %}
# from https://cipherli.st/ and https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html

ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
ssl_ecdh_curve secp384r1;
ssl_session_cache shared:SSL:10m;
ssl_session_tickets off;
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4 valid=300s;
resolver_timeout 5s;
add_header Strict-Transport-Security "max-age=63072000; includeSubdomains";
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;

ssl_dhparam /etc/ssl/certs/dhparam.pem;
{% endhighlight %}

Be careful about `add_header X-Frame-Options DENY;`, this setting will prevent your site from being embedded in `IFRAME`. Last task left for us before creating a new tunnel is site configurations. Replace `yourserver.example.com` with your own domain name. Taking port `80` just for port forwarding is not really helpful with nginx installed, let's use port `1111` instead.

{% highlight bash %}
$ sudo nano /etc/nginx/sites-available/default
{% endhighlight %}

{% highlight nginx %}
server {
    server_name yourserver.example.com;
    root /var/www/html;

    location / {
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   Host      $http_host;
        proxy_pass         http://127.0.0.1:1111;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Ssl on;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }

    listen 80;
    listen [::]:80;
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    include snippets/ssl-params.conf;
    ssl_certificate /etc/letsencrypt/live/yourserver.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourserver.example.com/privkey.pem;

    location ~ /.well-known {
         allow all;
    }
}
{% endhighlight %}

Reload the nginx configuration and try visiting your server with `https`. You should see green lock and `Bad Gateway`.

{% highlight bash %}
$ ssh -N user@yourserver.example.com -R 1111:localhost:8000
{% endhighlight %}

After you created the tunnel you should see the server you are running on your machine on port 8000. In case you don't have server ready you can start python server which serves files in current directory.

{% highlight bash %}
$ python -m SimpleHTTPServer 8000
{% endhighlight %}
