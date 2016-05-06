#SSL_HowTo

How to guide for enabling SSL and generating self-signed certificates for Ubuntu.

## Enable Mod SSL
`sudo a2enmod ssl`
## Generate a key (You will be prompted to create a pass phrase, setting one will require you enter the passphrase anytime the secure service is restart.  Not setting one is more convienent, but less secure)
`sudo openssl genrsa -des3 -out server.key 2048`
## Create insecure key
`sudo openssl rsa -in server.key -out server.key.insecure`
## Reverse the key names
```bash
sudo mv server.key server.key.secure
sudo mv server.key.insecure server.key
```
## Create a Certificate Signing Request (CSR)
`sudo openssl req -new -sha256 -key server.key -out server.csr`
## Use the CSR above to submit to a CA or proceed to self signing below
`sudo openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt`
-Country [US]
-State/Province [District of Columbia]
-Locality [Washington]
-Organization [GWU Libraries]
-Organizational Unit [Scholarly Technology Group]
-Common Name [DNS Name]
-Email Address [gwlib-root@groups.gwu.edu]
## Install SSL certs
```bash
sudo cp server.crt /etc/ssl/certs
sudo cp server.key /etc/ssl/private
```
## Configure Virtual Host
`sudo cp /etc/apache2/sites-available/default-ssl /etc/apache2/sites-available/host-ssl`
## Configure the following in your virtual host file
```apache2
<VirtualHost *:443>
ServerName example.com:443
```
## Add the following lines to your virtual host file
```apache2
SSLEngine on
SSLProtocol ALL -SSLv2 -SSLv3
SSLHonorCipherOrder On
SSLCipherSuite ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
SSLCertificateFile /etc/ssl/server.crt
SSLCertificateKeyFile /etc/ssl/server.key
```
##Enable the virtual host file
`sudo a2ensite virtual-host`
## (Optional) Add the following to your .htaccess file to rewrite traffic to http to https
```apache2
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule (.*) https://%{HTTP_HOST}%{REQUEST_URL} [R=301,L]
```
## Restart Apache2
`sudo service apache2 restart`
## Test and verify your conf
