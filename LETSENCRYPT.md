Atur virtualhost:80 dengan config berikut:
```
# Jangan izinkan proxypass .well-known
ProxyPass /.well-known !

# Alias untuk .well-known
Alias /.well-known/acme-challenge /home/muhipa/domains/v2.muhipa.web.id/public_html/.well-known/acme-challenge

<Directory /home/muhipa/domains/v2.muhipa.web.id/public_html/.well-known/acme-challenge>
    AllowOverride None
    Options None
    Require all granted
</Directory>
```
