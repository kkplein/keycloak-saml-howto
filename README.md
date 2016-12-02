# keycloak-saml-howto
A simple howto for keycloak / saml with external SP

This is a simple howto, to:
* setup RedHat's keycloak on debian jessie
* configure apache as an SSL reverse proxy
* connect it to your active directory DCs, using HAProxy failover
* configure kerberos for AD SSO
* connect some common clients to this setup using SAML

## setup java
Since keycloak is a java program, we need to install a java runtime engine, jre. Debian comes with openjdk-8-jre, but in my testing it did not work with kerberos authentication, so we are using oracle java 8, installed using these instructions: http://www.webupd8.org/2014/03/how-to-install-oracle-java-8-in-debian.html

Make sure to also configure oracle as the default runtime, so that your output is similar to this:
```
root@keycloak:~# java -version
java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
root@keycloak:~# 
```
## install keycloak
Download the standalone server distribution keycloak from http://www.keycloak.org/downloads.html and move the extracted files to /usr/local/keycloak. Start keycloak:
```
/usr/local/keycloak/bin/standalone.sh
```

## configure apache2 reverse proxy config
Install apache2 plus enable the required packages: 
```
a2enmod proxy proxy_http headers
```
A sample keycloak VirtualHost config:
```
<VirtualHost *:443>
		ServerAdmin webmaster@localhost
		ServerName keycloak.company.com
		DocumentRoot /var/www/html

    ProxyPreserveHost       On
    ProxyVia                Off
    ProxyRequests           Off
    ProxyPass               /       "http://localhost:8080/"
    ProxyPassReverse        /       "http://localhost:8080/"

    RequestHeader set X-Forwarded-Proto "https"
    RequestHeader set X-Forwarded-Port "443"


<Proxy *>
    Order deny,allow
    Allow from all
</Proxy>

		# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
		# error, crit, alert, emerg.
		# It is also possible to configure the loglevel for particular
		# modules, e.g.
		LogLevel info ssl:warn

		ErrorLog ${APACHE_LOG_DIR}/keycloak-error.log
		CustomLog ${APACHE_LOG_DIR}/keycloak-access.log combined


		#   SSL Engine Switch:
		#   Enable/Disable SSL for this virtual host.
		SSLEngine on


		SSLCertificateFile	/etc/ssl/apache2/keycloak/cert.pem
		SSLCertificateKeyFile /etc/ssl/apache2/keycloak/cert.key
		SSLCertificateChainFile /etc/ssl/apache2/keycloak/fullchain.pem

</VirtualHost>
```
## kerberos samba4
create a user for keycloak:
```
samba-tool user create keycloak --random-password
samba-tool spn add HTTP/keycloak.company.com keycloak
samba-tool domain exportkeytab --principal HTTP/keycloak.company.com keycloak.keytab
```
List keys in files.keytab:
```
klist -k ./keycloak.keytab 
 Keytab name: FILE:./keycloak.keytab
 KVNO Principal
 ---- --------------------------------------------------------------------------
   2 HTTP/keycloak.company.com@SAMBA.COMPANY.COM
   2 HTTP/keycloak.company.com@SAMBA.COMPANY.COM
   2 HTTP/keycloak.company.com@SAMBA.COMPANY.COM
```
Make sure that a forward AND reverse dns for keycloak.company.com exists!
