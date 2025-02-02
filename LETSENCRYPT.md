Atur virtualhost:80 dengan config berikut:
```
# Jangan izinkan proxypass .well-known
ProxyPass /.well-known !

# Alias untuk .well-known
Alias /.well-known/acme-challenge /home/reqishere/public_html/.well-known/acme-challenge

<Directory /home/reqishere/public_html/.well-known/acme-challenge>
    AllowOverride None
    Options None
    Require all granted
</Directory>
```
