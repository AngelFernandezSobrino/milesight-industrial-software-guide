# milesight-industrial-software-guide
Procedures for setting up the Milesight DeviceHub and VPN for Industrial Remote Control and IoT Applications

## 1 - Install certbot with Snap

```bash
# Update SNAP

sudo snap install core; sudo snap refresh core

# Remove other certbot packages

sudo apt remove certbot

# Install certbot

sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

## 2 - Get certificate for the desired domain

### 2.1 - If port 80 is free and has not to be used

```

# Execute standalone http server for certificate checking

sudo certbot certonly --standalone -d <your.domain> -m <registration@email>
```

### 2.2 - If port 80 is configured on the application 
```

# Use webroot directoy for certificate validation

sudo certbot certonly --webroot -d <your.domain> -m <registration@email> -w </your/path>
```

With Devicehub is configured to use port 80 by default, so the use of webroot is recomended. Webroot folder is by default /clouddata/www/cloud_management

With MilesightVPN, the use of webroot (which should be /milesight_vpn directory) has not been tested, so the standalone option is the recomended way. The default ports for the frontend are defined to 18080 and 18443. It is recomeded to change the ssl port to 443 at /milesight_vpn/server/server.js at line 18.


## 3 - Create symlink to nginx folder

### Devicehub: 

The directory /clouddata/www/cloud_management/certs should be empty.

Force DeviceHub to use a certificate using the desired domain, at the web frontend in configuration / Domain. Input the desired domain and two empty or random files at Certificate and Private Key. The process will fail in that conditions, it is normal, now DeviceHub is configured to use SSL certificates.

At the directoy /clouddata/www/cloud_management/certs and /clouddata/server/nginx/ssl the files nginx.crt and nginx.key should appear. Change the name of the files as backup. Then create a symlink from the certificates to /clouddata/www/cloud_management/certs and /clouddata/server/nginx/ssl:

'''bash
mv /clouddata/www/cloud_management/certs/nginx.crt /clouddata/www/cloud_management/certs/nginx.crt.backup
mv /clouddata/www/cloud_management/certs/nginx.key /clouddata/www/cloud_management/certs/nginx.key.backup
mv /clouddata/server/nginx/ssl/domain.crt /clouddata/server/nginx/ssl/domain.crt.backup
mv /clouddata/server/nginx/ssl/domain.key /clouddata/server/nginx/ssl/domain.key.backup

ln -s /etc/letsencrypt/live/<domainname>/fullchain.pem /clouddata/www/cloud_management/certs/nginx.crt
ln -s /etc/letsencrypt/live/<domainname>/privkey.pem /clouddata/www/cloud_management/certs/nginx.key


ln -s /etc/letsencrypt/live/<domainname>/fullchain.pem /clouddata/server/nginx/ssl/domain.crt
ln -s /etc/letsencrypt/live/<domainname>/privkey.pem /clouddata/server/nginx/ssl/domain.key
'''

### MilesightVPN

In this case the files are already on the server, just the creation of the symlink is required:

'''bash
rm /milesight_vpn/server/https/server.crt.backup
rm /milesight_vpn/server/https/server.key.backup

ln -s /etc/letsencrypt/live/<domainname>/fullchain.pem /milesight_vpn/server/https/server.crt
ln -s /etc/letsencrypt/live/<domainname>/privkey.pem /milesight_vpn/server/https/server.key
'''

Restart the VM to make changes take effect.

## Milesight DeviceHub NAT problems

In case that your server has not direct public IP address on its own NIC, and it can by reached by DMZ or NAT, the public IP where the server would be listening has to be configured in the Public IP field at the DeviceHub configuration page (also enabling NAT option). If the public IP is not fixed, or if, for reliability, is desired to use a DNS subdomain, the deviced hub database and the frontend have to be tunned to allow that (which is not possible by default).

Its a work-around proposed by Milesight IoT Support service, it has been tested.

```bash

# Access to the mysql database

cd /clouddata/server/mysql/bin
./mysql -uroot -padmin

```

```sql

-- Modify tables to change IP column to varchar(255) value to be able to handle a host name string

ALTER TABLE `ursalink_myacs`.`tbl_acs_ssh_tunnel_port` MODIFY COLUMN `ip` varchar(255) CHARACTER SET latin1 COLLATE latin1_swedish_ci NOT NULL FIRST;
ALTER TABLE `ursalink_myacs`.`tbl_acs_ssh_tunnel` MODIFY COLUMN `ip` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL COMMENT 'server ssh IP address' AFTER `id`;
ALTER TABLE `ursalink_myacs`.`tbl_acs_config` MODIFY COLUMN `server_ip` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL FIRST;
quit;                                          // Leave database
```
```bash

# Access general.php front UI file

vim /clouddata/www/cloud_management/application/views/general.php
```
On this file go to line 144 and delete data-type="ip" and change maxlength to 128.

Go to configuration in Devicehub and set the Host name at the NAT Section.