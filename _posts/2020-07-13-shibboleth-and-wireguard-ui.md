---
layout: post
title:  "Shibboleth and Wireguard UI"
date:   2020-07-13
---

Cross-post from [wg-ui#38](https://github.com/EmbarkStudios/wg-ui/issues/38#issuecomment-657375867).

This is a write-up how [Stockholm University](https://www.su.se) protected our
[Wireguard UI](https://github.com/EmbarkStudios/wg-ui) with a [Shibboleth
SP](https://wiki.shibboleth.net/confluence/display/SP3/Home) and [Apache
httpd](https://httpd.apache.org). I will not cover how to configure `shibd` or
the IdP part of this integration.

The Univerity is heavly in to [Single
sign-on](https://en.wikipedia.org/wiki/Single_sign-on) and
[SAML](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) so
`shibd` is one of the more common tools we have and use. Together with `apache`
it's easy to create SSO for application that can't speak native SAML. The
combination `shibd` and `apache` handles all the authentication and in this
case even a rough authorization (more on that later) and proxies the request to
the service.

Most SAML attributes in the .edu world are based on LDAP attributes.
[eduPersonPrincipalName](https://www.internet2.edu/media/medialibrary/2013/09/04/internet2-mace-dir-eduperson-201203.html#eduPersonPrincipalName)
(or eppn as Shibboleth calls it) is our primary key to identify users so that
is released from the IdP to the SP as a SAML attribute and then forward/proxied
as request header to the application. The only thing that needs to be
configured in the Wireguard UI end is that the application needs to be started
with the `--auth-user-header` flag set to `eppn`.

### The `apache` configuration

```
<VirtualHost *:443>
    <LocationMatch "/">
        AuthType Shibboleth
        Require shib-attr entitlement ~ ^urn:mace:swami.se:gmai:vpn:user$
        ShibRequireSessionWith idp.example.com
        ShibUseHeaders On
    </LocationMatch>

    SSLCertificateFile    /path/to/vpn.example.com.pem
    SSLCertificateKeyFile /path/to/vpn.example.com.key
    SSLCertificateChainFile /path/to/DigiCertCA-2024-11-18.crt

    ProxyPass "/" "http://127.0.0.1:8080/"
    ProxyPassReverse "/" "http://127.0.0.1:8080/"
</VirtualHost>
```

#### Configuration in depth

```
Require shib-attr entitlement ~ ^urn:mace:swami.se:gmai:su-vpn:user$
```
We have alot of users at the University and not all of them are eligible to use
Wireguard UI. By default apache and shibd lets everyone through and since
Wireguard UI has no knowlege about the user in beforehand we release another
([eduPersonEntitlement](https://www.internet2.edu/media/medialibrary/2013/09/04/internet2-mace-dir-eduperson-201203.html#eduPersonEntitlement))
from the IdP to the SP and require a specific value on the user in order to be
allowed to use the service.

```
ShibUseHeaders On
```

This enables `shibd` to publish SAML attributes to the application (in our case
proxy) through request headers.

---

Thats is! I hope it could be useful someone else. The setup works flawless and
big thanks to [EmbarkStudios](https://www.embark.dev) for a great application.
