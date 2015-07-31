# Nginx SSL Config

## set up the certificate bundle
Generate the certificate request the same way as with Apache. You will receive three documents back from Incommon, A certificate and two bundes:
* X509 Certificate only, Base64 encoded
* X509 Intermediates/root only, Base64 encoded
* as X509 Intermediates/root only Reverse, Base64 encoded

Concatenate the certificate only file to one of the intermediate bundles (reverse, base64 encoded is recommended). Insert a newline between the files, or you will receive an bad end line error from Nginx:
```bash
cat certfile.cer <(echo) bundlefile.cer > yournewcertbundle.cer
```

Place the Certificate in your `/etc/ssl/certs` directory.

## Edit your Nnginx host file
Add the following directives in the server stanza:
```nginx
server {
        #...omited section
        listen 443 ssl;
        ssl_prefer_server_ciphers on;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers kEECDH+AESGCM+AES128:kEECDH+AES128:kRSA+AESGCM+AES128:kRSA+AES128:!RC4:!aNULL:!eNULL:!MD5:!EXPORT:!LOW:!SEED:!CAMELLIA:!IDEA:!PSK:!SRP:!SSLv2;
        
        ssl_certificate /etc/ssl/certs/yournewcertbundle.cer;
        ssl_certificate_key /etc/ssl/private/yourkeyfile.key;
        #...omited section
        }
```
Restart nginx with `sudo service nginx restart`. If there nginx fails, check `/var/log/nginx/error.log`. 
