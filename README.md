# Matrix chat server on OpenBSD

Run a Matrix homeserver on OpenBSD with PostgreSQL server. This documents installing either:

 - [Synapse](https://github.com/matrix-org/synapse) the Matrix reference homeserver written in python
 - [Dendrite](https://github.com/matrix-org/dendrite) the Matrix second-generation Matrix homeserver written in Go

*Note:*

 * Replace `server.chat` with your domain name in all config and commands.
 * PostgreSQL is remote to the dendrite server.
   - You will need postgres:
     * hostname
     * username and password
     * database name
     * postgres `pg_hba.conf` configured for your server to connect

## Install pre-requisites

```shell
pkg_add nginx certbot
```

## Nginx

Setup nginx with initial config [`nginx.initial.conf`](./nginx.initial.conf) in `/etc/nginx/nginx.conf`

Make directory for Lets Encrypt challenge:

```shell
mkdir -p /var/www/letsencrypt/.well-known/acme-challenge
```

Point you domain A record to the IP of the server.

Start nginx it will listen on port `80`:

```shell
rcctl start nginx
```

Test you can hit the server on http://server.chat/.well-known/acme-challenge/test.html and you will get a `200`.

```shell
$ echo "working" > /var/www/letsencrypt/.well-known/acme-challenge/test.html
$ curl -I http://server.chat/.well-known/acme-challenge/test.html

HTTP/1.1 200 OK
Server: nginx
Date: Mon, 01 Feb 2021 02:59:17 GMT
Content-Type: text/html
Content-Length: 8
Last-Modified: Mon, 01 Feb 2021 02:55:58 GMT
Connection: keep-alive
ETag: "60176dbe-8"
Accept-Ranges: bytes
```

Delete the test

```shell
rm /var/www/letsencrypt/.well-known/acme-challenge/test.html
```

## Certbot

Edit the base config in `/etc/letsencrypt/cli.ini`

```
rsa-key-size = 4096
email = webmaster@server.chat
text = True
authenticator = webroot
webroot-path = /var/www/letsencrypt
```

Request cert:

```shell
$ certbot certonly -d server.chat

Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator webroot, Installer None
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for server.chat
Using the webroot path /var/www/letsencrypt for all unmatched domains.
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/server.chat/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/server.chat/privkey.pem
   Your cert will expire on 2021-05-02. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot
   again. To non-interactively renew *all* of your certificates, run
   "certbot renew"
```

Generate a dhparam file for NGINX:

```
openssl dhparam -out /etc/nginx/ssl/dhparam.pem 2048
```

## Nginx TLS

Setup nginx with full config [`nginx.full.conf`](./nginx.full.conf) in `/etc/nginx/nginx.conf`, pointing the following three files to the cert, key and dhparams you just generated:

```nginx
ssl_certificate /etc/letsencrypt/live/server.chat/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/server.chat/privkey.pem;
ssl_dhparam /etc/nginx/ssl/dhparam.pem;
```

Reload nginx:

```shell
rcctl reload nginx
```

Check the secured response

```shell
$ curl https://server.chat/.well-known/matrix/server

{ "m.server": "server.chat:443" }
```

## Service user setup

Create user

```shell
adduser -group wheel -class daemon -batch matrix
```

# Homeserver choice

From here you can install and configure:

 - [Synapse](./synapse/)
 - [Dendrite](./dendrite/)