# keycloak-saml-howto
A simple howto for keycloak / saml with external SP

This is a simple howto, to:
* setup RedHat's keycloak on debian jessie
* configure apache as an SSL reverse proxy
* connect your active directory DCs with HAProxy failover
* configure kerberos for AD SSO
* connect simplesamlphp to this setup as a SAML client
* connect wordpress to this setup as a SAML client

```
piece of code
```
## java
Since keycloak is a java program, we need to install a java runtime engine, jre. Debian comes with openjdk-8-jre, but in my testing it did not work with kerberos authentication, so we are using oracle java 8, installed using these instructions: http://www.webupd8.org/2014/03/how-to-install-oracle-java-8-in-debian.html
Make sure to also configure oracle as the default runtime, so that this works:
```
root@keycloak:~# java -version
java version "1.8.0_111"
Java(TM) SE Runtime Environment (build 1.8.0_111-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.111-b14, mixed mode)
root@keycloak:~# 
```
## keycloak
