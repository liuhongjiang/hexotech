title: Enable https and tokenless auth for keystone
date: 2016-12-23 19:48:53
tags:
categories: keystone, openstack
---

## enable https for keystone

actually it's quite easy, first you need load mod_ssl module of apache,
then only thing you need to do is change the [wsgi-keystone.conf](https://github.com/openstack/keystone/blob/master/httpd/wsgi-keystone.conf):

like this:
```
<VirtualHost *:5000>
    ...
    SSLEngine on
    SSLCertificateKeyFile /pass/to/key-file.pem
    SSLCertificateFile /path/to/server.cer
    ...
</VirtualHost>
```

if your server certificate is not signed by the root CA, then you need the intermediate CA certificates.
please notice `SSLCertificateChainFile` became obsolete with version 2.4.8, when SSLCertificateFile was extended to also load intermediate CA certificates from the server certificate file.

<!--more-->

## how to using openstack clint to connect to keystone with our own server certificate

with our own server certificate, the openstack client will report the following error

```
[andrew@localhost certifi]$ openstack user list
Discovering versions from the identity service failed when creating the password plugin. Attempting to determine version from URL.
SSL exception connecting to https://test-os.andrew.com:5000/v3/auth/tokens: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed (_ssl.c:765)
```

then you add your own CA certificate following [this article](/hexotech/2016/12/23/setup-your-own-ca/#for-python-requests-to-add-ca), it can be succeeded:

```
[andrew@localhost kolla]$ openstack user list
+----------------------------------+-------+
| ID                               | Name  |
+----------------------------------+-------+
| ce52a85aea0f46abb3930d9c8f866131 | admin |
+----------------------------------+-------+
```

## enable tokenless author

for this, please following the [manual of the openstack](http://docs.openstack.org/developer/keystone/configure_tokenless_x509.html).

There some point you need know, in the apache configuration:
```
SSLCACertificatePath /etc/apache2/capath
```

you need put the CA certificates of CA who sign the clinet certificates in that directory, and you need to generated a hash-value symbolic file for it.

please refer to the section `generate hash-value for apache SSLCACertificatePath` Directive for details.

When config the trusted_issuer and generate IdP ID of the issuer_dn, you can using the following way to find it out.

Frist you need have one 'trusted_issuer' in your keystone.conf, if you didn't, you can configure an arbitrary string, then using the curl command like this:

```
curl -v -k -s -X GET --cert /home/andrew/CA/certs/client.cer \
     --key /home/andrew/CA/private/client-key-no-passwrod.pem \
     -H "X-Project-Name: admin"      \
     -H "X-Project-Domain-Id: default"    \      
     -H "X-Subject-Token: dfaa8c530cd548c59ae60ddce4fc594c"      \
     https://test-os.andrew.com:5000/v3/auth/tokens
```

then you can find the issues in the log

```
2016-12-23 08:33:06.029 17 INFO keystone.middleware.auth [req-a7bcd1e9-77d7-44a6-be95-6d081680ef40 - - - - -] The client issuer CN=myname,OU=mygroup,O=myorganization,L=mycity,ST=myprovince,C=CN does not match with the trusted issuer ['emailAddress=john@openstack.com,CN=john,OU=keystone,O=openstack,L=Sunnyvale,ST=California,C=US']
```

the `CN=myname,OU=mygroup,O=myorganization,L=mycity,ST=myprovince,C=CN` is the issue dn you can user for trusted_issuer and generate IdP ID.

## generate hash-value for apache SSLCACertificatePath Directive

http://httpd.apache.org/docs/current/mod/mod_ssl.html#SSLCACertificatePath

So usually you can't just place the Certificate files there: you also have to create symbolic links named hash-value.N. And you should always make sure this directory contains the appropriate symbolic links.

in this article, it explained the reason:

https://blog.g3rt.nl/client-certificate-authentication-apache.html

Suppose you have hundreds of these certificates, then Apache would need to open every certificate until it finds the right one. You can imagine that would be very inefficient. To speed that up, Apache looks for a file with the hash of the certificate it gets from the client. For example, if my certificate would be hashed as 27e66395 then it would look for files with the name of 27e66395.X where X is a number starting with 0.

below command, you can find the hash value from the ca file.

```
openssl x509 -noout -hash -in NAME-OF-CA-FILE
```

## Reference

[using curl to requst keystone api](http://www.florentflament.com/blog/setting-keystone-v3-domains.html)
