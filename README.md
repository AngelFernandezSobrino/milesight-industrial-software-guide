# milesight-industrial-software-guide
Procedures for setting up the Milesight DeviceHub and VPN for Industrial Remote Control and IoT Applications



install certbot with Snap

certbot certonly

Option 2 (Webroot)

Input webroot: /clouddata/www/cloud_management

Create symlink to nginx folder

Force DeviceHub to import a certificate, using the desired domain, with empty or random txt files as certificates. It will fail, it is normal, the files are not real certificates, we just want to configure DeviceHub to use SSL Certificates.

Befaore that, the directoy /clouddata/www/cloud_management/certs should be empty. With the previous action the files nginx.crt and nginx.key should appear.

Create a symlink from the certificates to /clouddata/www/cloud_management/certs and /clouddata/server/nginx/ssl, remove previous existing certificates:

'''bash
rm /clouddata/www/cloud_management/certs/nginx.crt
rm /clouddata/www/cloud_management/certs/nginx.key
rm /clouddata/server/nginx/ssl/domain.crt
rm /clouddata/server/nginx/ssl/domain.key

ln -s /etc/letsencrypt/live/<domainname>/fullchain.pem /clouddata/www/cloud_management/certs/nginx.crt
ln -s /etc/letsencrypt/live/<domainname>/privkey.pem /clouddata/www/cloud_management/certs/nginx.key


ln -s /etc/letsencrypt/live/<domainname>/fullchain.pem /clouddata/server/nginx/ssl/domain.crt
ln -s /etc/letsencrypt/live/<domainname>/privkey.pem /clouddata/server/nginx/ssl/domain.key
'''

