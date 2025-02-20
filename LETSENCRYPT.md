1.    Atur virtualhost:80 dengan config berikut:
```
# Jangan izinkan proxypass .well-known
ProxyPass /.well-known !

# Alias untuk .well-known
Alias /.well-known/acme-challenge /home/{ NAMA USER }/public_html/.well-known/acme-challenge

<Directory /home/{ NAMA USER }/public_html/.well-known/acme-challenge>
    AllowOverride None
    Options None
    Require all granted
</Directory>

```

2.    gunakan certbot certonly --dry-run:
```
sudo certbot certonly --dry-run -d nama_domain.com -w /home/{ NAMA USER }/public_html
```

3.    bisa coba buat file tes untuk curl lewat terminal:
```
echo "ini isi file testing.txt" > /home/{ NAMA USER }/public_html/.well-known/acme-challenge/testing.txt
curl http://nama_domain.com/.well-known/acme-challenge/testing.txt
```

4.    jika berhasil, coba pakai virtualmin untuk buat SSL yaitu dengan cara:
- login
- masuk situs yang ingin diatur
- buka menu "Manage Virtual Server" lalu klik "Setup SSL Certificate"
- pilih tab "Let's Encrypt"
- atur domain yang ingin diatur
- jika sudah, klik "Request Certificate"
- Tunggu hingga selesai...

5.    jika dapat error "ssl_error_rx_record_too_long", cek config VirtualHost untuk port 443:
- login
- masuk ke webmin config
- masuk ke config "Apache Webserver"
- masuk ke "Global Configuration"
- pilih menu "Edit Config Files"
- cek VirtualHost port :443, set agar bukan ip address yang ada di sana melainkan *. contoh:
```
<VirtualHost 192.168.1.2:443>
    ServerName nama_domain.com
    ...
</VirtualHost>
```
menjadi:
```
<VirtualHost *:443>
    ServerName nama_domain.com
    ...
</VirtualHost>
```
- jika sudah di ubah, lakukan "Save" lalu ujicoba domain yang di ubah tadi.
