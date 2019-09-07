# Notes on SSL with Docker

Setting up SSL with Docker requires multiple steps. First, get your domain ready.

* Make sure that your DNS domain name is mapped to the docker host's IP address
* Get `certificate.crt`, `private.key`, and `ca_bundle.crt` files for your domain at [sslforfree.com](https://www.sslforfree.com/).

Second, make sure that you have the https link for `publicIp` in the `js/secondStateJS.js` file. You can also edit the `/var/www/html/js/secondStateJS.js` file inside Docker.

Third, add the following in the `config/site.conf` file. You can also edit the `/etc/apache2/site-enabled/search-engine.conf` file inside Docker.

```text
<VirtualHost *:443>
    ProxyPreserveHost On
    ProxyPass /api http://127.0.0.1:8080/api
    ProxyPassReverse /api http://127.0.0.1:8080/api
    ServerName localhost
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error-ssl.log
    CustomLog ${APACHE_LOG_DIR}/access-ssl.log combined

    SSLEngine on
    SSLCertificateFile /etc/apache2/certificate.crt
    SSLCertificateKeyFile /etc/apache2/private.key
    SSLCertificateChainFile /etc/apache2/ca_bundle.crt

    <Location "/">
        Header always set Access-Control-Allow-Origin "*"
        Header always set Access-Control-Allow-Methods "POST, GET, OPTIONS"
        Header always set Access-Control-Max-Age "1000"
        Header always set Access-Control-Allow-Headers "x-requested-with, Content-Type, origin, authorization, accept, client-security-token"
        RewriteEngine On
        RewriteCond %{REQUEST_METHOD} OPTIONS
        RewriteRule ^(.*)$ $1 [R=200,L]
    </Location>
</VirtualHost>
```

Fourth, build and then start Docker with the following command to turn on port 443.

```text
$ docker run -d -it --rm -p 80:80 -p 443:443 -v $HOME/.aws:/root/.aws search-engine
```

Fifth, start Docker and login. Create the `certificate.crt`, `private.key`, and `ca_bundle.crt` files under directory `/etc/apache2/`

Sixth, enable SSL modules.

```text
root# cd /etc/apache2/sites-enabled
root# ln -s ../mods-available/socache_shmcb.load socache_shmcb.load
root# ln -s ../mods-available/ssl.load ssl.load
root# ln -s ../mods-available/ssl.conf ssl.conf
```

Finally, restart Apache from inside Docker.

```text
root# apachectl restart
```

