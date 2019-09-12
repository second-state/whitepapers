# Notes on SSL with Docker

Setting up SSL with Docker requires multiple steps. First, get your domain ready.

* Make sure that your DNS domain name is mapped to the docker host's IP address
* Get `certificate.crt`, `private.key`, and `ca_bundle.crt` files for your domain at [sslforfree.com](https://www.sslforfree.com/).

Second, make sure that you have the https link for `publicIp` in the `js/secondStateJS.js` file. 

Third, add the following in the `config/site.conf` file. 

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
root# cd /etc/apache2/mods-enabled
root# ln -s ../mods-available/socache_shmcb.load socache_shmcb.load
root# ln -s ../mods-available/ssl.load ssl.load
root# ln -s ../mods-available/ssl.conf ssl.conf
```

Finally, restart Apache from inside Docker.

```text
root# apachectl restart
```

## Use Let's Encrypt

Alternatively, you can use Let's Encrypt to setup SSL automatically. Start Docker with the following command to turn on port 443.

```text
$ docker run -d -it --rm -p 80:80 -p 443:443 -v $HOME/.aws:/root/.aws search-engine
```

Log into Docker.

```bash
$ docker exec -it container_id bash
```

Next, use Let's Encrypt services to setup SSL.

```bash
$ apt update && apt upgrade
$ apt install wget
$ wget https://dl.eff.org/certbot-auto -O /usr/sbin/certbot-auto
$ chmod a+x /usr/sbin/certbot-auto
$ certbot-auto --apache -d search.domain.com
```

Next, please open the `/etc/apache2/sites-enabled/*-ssl.conf` file \(which was created automatically by the above command\) and add the following code inside the `VirtualHost` section.

```text
<VirtualHost *:443>
    ... ...
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

Replace the HTTP IP address below with your new HTTPS domain name.

* `ServerName` in apache config `config/site.conf`. 
* `publicIp` in `js/secondStateJS.js`.

Exit docker and give it a reboot.

```bash
$ docker restart container_id
```

Now, you should be able access the search engine from `https://search.domain.com` now.

