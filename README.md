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

You'll be prompted to answer some questions:
```
Country [US]
State/Province [District of Columbia]
Locality [Washington]
Organization [GWU Libraries]
Organizational Unit [Scholarly Technology Group]
Common Name [DNS Name]
Email Address [gwlib-root@groups.gwu.edu]
```
Leave the challange section blank in most cases.
## Get a signature for your certificate.
If you're getting a proper certificate, send the csr file to an authrority and they will send back a certificate. at GWU, email it to ithelp@gwu.edu.
### Self Signing
If you're working in development and would prefer to sign the certificate yourself, do the following
```bash
sudo openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```
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
...
SSLEngine on
SSLProtocol ALL -SSLv2 -SSLv3
SSLHonorCipherOrder On
SSLCipherSuite ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
SSLCertificateFile /etc/ssl/certs/server.crt
SSLCertificateKeyFile /etc/ssl/private/server.key
...
</VirtualHost>
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

## InCommon Intermediary
If you're using a certificate signed by in-common, download their intermediary bundle and include it in your chain. The bundle is available at `http://www.incommon.org/certificates/repository/sha384%20Intermediate%20cert.txt`. Rename this certificate and move it to `/etc/ssl/intermediate/incommon-ssl.ca-bundle`. Then add the following directive after you certificate and key files: `/etc/ssl/intermediate/incommon-ssl.ca-bundle`.
