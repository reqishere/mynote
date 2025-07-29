# Default Conf yang perlu digunakan:

{Input Chain}
(default rules to keep)
```
/ip firewall filter
add action=accept chain=input comment="defconf: accept established,related,untracked" connection-state=established,related,untracked
add action=drop chain=input comment="defconf: drop invalid" connection-state=invalid
add action=accept chain=input comment="defconf: accept ICMP" protocol=icmp
add action=accept chain=input comment="defconf: accept to local loopback (for CAPsMAN)" dst-address=127.0.0.1
```

{forward Chain}
(default rules to keep)
```
/ip firewall filter
add action=accept chain=forward comment="defconf: accept in ipsec policy" ipsec-policy=in,ipsec
add action=accept chain=forward comment="defconf: accept out ipsec policy" ipsec-policy=out,ipsec
add action=fasttrack-connection chain=forward comment="defconf: fasttrack" connection-state=established,related
add action=accept chain=forward comment="defconf: accept established,related, untracked" connection-state=established,related,untracked
add action=drop chain=forward comment="defconf: drop invalid" connection-state=invalid
```
# Linux Server Push SSL Sertificate from Let's Encrypt

Mengapa Sistem "Push" Adalah Solusi Terbaik Anda
Sistem "push" memanfaatkan fakta bahwa server lokal Anda sudah memiliki IP publik dan dapat mengelola sertifikat Let's Encrypt dengan sukses. Prosesnya adalah sebagai berikut:

Server Lokal (dengan IP Publik) Mengelola Sertifikat: Server Anda akan terus berfungsi sebagai titik utama untuk memperbarui sertifikat Let's Encrypt (misalnya dengan Certbot).
Ekspor Sertifikat dan Private Key: Setelah sertifikat berhasil diperbarui, skrip di server Anda akan mengekspor file sertifikat (fullchain.pem atau .crt) dan private key (privkey.pem atau .key).
Transfer Aman ke Mikrotik: Skrip akan mengirimkan file-file ini ke Mikrotik melalui protokol yang aman seperti SCP (Secure Copy Protocol) atau SFTP (SSH File Transfer Protocol).
Impor dan Konfigurasi di Mikrotik: Setelah file sampai di Mikrotik, skrip lokal di Mikrotik akan mengimpor sertifikat dan private key, lalu mengkonfigurasi profil Hotspot untuk menggunakan sertifikat baru tersebut.

Langkah-langkah Implementasi Sistem "Push" Otomatis
Ini akan melibatkan scripting di sisi server dan juga di Mikrotik.

A. Persiapan di Mikrotik (Sekali Saja)
Buat User Khusus untuk Otomatisasi:

Ini sangat penting untuk keamanan. Jangan gunakan user admin Anda.
Buka Winbox, pergi ke System > Users.
Klik "+" untuk menambah user baru.
Name: automation_cert (atau nama lain yang jelas).
Group: Buat group baru dengan izin minimal yang diperlukan:
  Pergi ke tab Groups.
  Klik "+".
  Name: cert_automation_group
  Policy: Hanya centang ftp, reboot (jika perlu reboot setelah update, jarang), policy (untuk mengubah config sertifikat/hotspot), read, write, ssh. Ini cukup untuk memungkinkan user mengunggah file dan menjalankan perintah.
  Password: Atur password yang kuat untuk user automation_cert.
  Allowed Address: (Opsional, tapi sangat disarankan) Jika server Anda memiliki IP statis, masukkan IP tersebut di sini untuk membatasi akses SSH/FTP hanya dari server Anda.

Pastikan SSH Server Aktif di Mikrotik:

Pergi ke IP > Services.
Pastikan ssh dalam keadaan enabled. Default port adalah 22.
Buat Skrip Impor Sertifikat di Mikrotik:
  Ini adalah skrip yang akan dieksekusi oleh server setelah file sertifikat diunggah.

Buka Winbox, pergi ke System > Scripts.
Klik "+".
Name: import_hotspot_cert
Source: Masukkan kode berikut. (Ganti nama_sertifikat_anda dengan nama yang Anda inginkan untuk sertifikat yang diimpor, misal your_domain.com).

{your script source in mikrotik scripts}
```
# Hapus sertifikat dengan nama yang sama jika sudah ada, untuk menghindari duplikat
# Ini penting jika Anda selalu ingin sertifikat terbaru menjadi "your_domain.com"
:local newCertName "your_domain.com";
:local existingCertId [/certificate find where name="$newCertName"];
:local existingCertId1 [/certificate find where name="$newCertName_1"];
:local fullchainFileName "fullchain.pem";
:local privkeyFileName "privkey.pem";

:if ([:len $existingCertId] > 0) do={
    /certificate remove $existingCertId;
    /certificate remove $existingCertId1;
    :log info "Mikrotik: Removed existing certificate named $newCertName.";
}

:if ([:len $existingCertId1] > 0) do={
    /certificate remove $existingCertId1;
    :log info "Mikrotik: Removed existing certificate named $newCertName _1.";
}

:delay 3s;


# Impor fullchain. Akan otomatis mencocokkan dengan private key yang sudah ada
# Gunakan nama yang sama agar keduanya terikat pada satu entri sertifikat
/certificate import name="$newCertName" file-name="$fullchainFileName" passphrase="";
:delay 2s;
:log info "Mikrotik: Fullchain certificate $fullchainFileName imported for $newCertName fullchain cert.";


# Impor private key terlebih dahulu. Pastikan nama file sesuai dengan yang di-upload
/certificate import name="$newCertName" file-name="$privkeyFileName" passphrase="";
:delay 2s;
:log info "Mikrotik: Private key $privkeyFileName imported as $newCertName private cert.";

###########################

:local certValidAndTrusted [/certificate find name="$newCertName"];
:local countCertValid [:len $certValidAndTrusted];
:local profileHotspot "Profile SMK";

:delay 2s;

# Cari sertifikat baru berdasarkan nama yang sudah kita berikan saat import
:log info "Mikrotik: cek sertifikat valid $countCertValid";
:if ($countCertValid=1) do={
    /ip hotspot profile set [find name=$profileHotspot] ssl-certificate=$newCertName;
    :log info "Mikrotik: Hotspot profile updated with new certificate: $newCertName";

    # Hapus file mentah setelah diimpor (untuk keamanan)
    /file remove $privkeyFileName;
    /file remove $fullchainFileName;
    :log info "Mikrotik: Certificate files removed from flash.";
} else={
    :log error "Mikrotik: New certificate named '$newCertName' not found or not valid after import. Hotspot profile not updated.";
}
```
click "Apply" lalu "OK"


B. Persiapan di Server Lokal (Linux dengan Certbot)
Asumsi Anda menggunakan Certbot untuk Let's Encrypt.

Identifikasi Lokasi Sertifikat:
Sertifikat Let's Encrypt untuk example.com biasanya ada di /etc/letsencrypt/live/example.com/.
Anda akan membutuhkan fullchain.pem (sertifikat + intermediate) dan privkey.pem (private key).

Instal SCP Client (jika belum ada):
Umumnya ssh dan scp sudah terinstal di kebanyakan distribusi Linux.


disini, kita perlu buat 2 script, script untuk auto renew cert pakai certbot, dan script untuk eksekusi auto push cert dari server ke mikrotik. nanti script certbot harus atur --deploy-hook yang ditujukan ke script push cert, karena pembuatan cert tidak langsung bisa tanpa ada jeda.

dibawah pembuatan file untuk auto certbot renew:
Buat file skrip di server Anda di folder ini (misal di /opt/scripts/):
Buat Skrip Otomatisasi (misal: automatic_certbot_file.sh):

{/opt/scripts/automatic_certbot_file.sh}
```
#!/bin/bash

certbot renew --quiet --non-interactive --deploy-hook "/opt/scripts/push_cert_to_mikrotik.sh"
```
Pastikan skrip ini memiliki izin eksekusi (chmod +x automatic_certbot_file.sh).

Buat file skrip di server Anda di folder ini (misal di /opt/scripts/):
Buat Skrip Otomatisasi (misal: push_cert_to_mikrotik.sh):
Ganti MIKROTIK_IP, MIKROTIK_USER, dan MIKROTIK_PASS dengan detail Mikrotik Anda.
Ganti DOMAIN dengan domain Anda.

{/opt/scripts/push_cert_to_mikrotik.sh}
```
#!/bin/bash

# --- Konfigurasi ---
DOMAIN="your_domain"
CERT_DIR="/etc/letsencrypt/live/$DOMAIN"
MIKROTIK_IP="192.168.88.1" # Contoh: 192.168.88.1
MIKROTIK_SSH_PORT="22"
MIKROTIK_USER="automation_cert"
MIKROTIK_PASS="[PASSWORD_USER_AUTOMATION_CERT]" # HATI-HATI DENGAN PLAIN TEXT PASSWORD
SSH_PRIVATE_FILE="/home/your_user/.ssh/id_rsa"

# --- File Sertifikat ---
FULLCHAIN_FILE="$CERT_DIR/fullchain.pem"
PRIVKEY_FILE="$CERT_DIR/privkey.pem"

# --- Transfer File ke Mikrotik via SCP ---
echo "Uploading certificate files to Mikrotik..."
# scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "$FULLCHAIN_FILE" "$MIKROTIK_USER@$MIKROTIK_IP:fullchain.pem"
scp -i "$SSH_PRIVATE_FILE" -P "$MIKROTIK_SSH_PORT" -o StrictHostKeyChecking=no "$FULLCHAIN_FILE" "$MIKROTIK_USER@$MIKROTIK_IP:fullchain.pem"
if [ $? -ne 0 ]; then
    echo "Error: Failed to upload fullchain.pem. Exiting."
    exit 1
fi
# scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "$PRIVKEY_FILE" "$MIKROTIK_USER@$MIKROTIK_IP:privkey.pem"
scp -i "$SSH_PRIVATE_FILE" -P "$MIKROTIK_SSH_PORT" -o StrictHostKeyChecking=no "$PRIVKEY_FILE" "$MIKROTIK_USER@$MIKROTIK_IP:privkey.pem"
if [ $? -ne 0 ]; then
    echo "Error: Failed to upload privkey.pem. Exiting."
    exit 1
fi
echo "Files uploaded successfully."

# --- Eksekusi Skrip Impor di Mikrotik via SSH ---
echo "Executing import script on Mikrotik..."
# ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "$MIKROTIK_USER@$MIKROTIK_IP" "/system script run import_hotspot_cert"
ssh -i "$SSH_PRIVATE_FILE" -p "$MIKROTIK_SSH_PORT" -o StrictHostKeyChecking=no "$MIKROTIK_USER@$MIKROTIK_IP" "/system script run import_hotspot_cert"
if [ $? -ne 0 ]; then
    echo "Error: Failed to execute script on Mikrotik."
    exit 1
fi
echo "Mikrotik certificate update completed."

# --- REKOMENDASI KEAMANAN TINGGI: Gunakan SSH Key Authentication ---
# Ini jauh lebih aman daripada menyimpan password di skrip.
# 1. Buat SSH key pair di server Anda: `ssh-keygen` (biasanya di ~/.ssh/id_rsa dan ~/.ssh/id_rsa.pub)
# 2. Salin public key (`id_rsa.pub`) ke Mikrotik:
#    Di Winbox, buka System -> Users -> Pilih user 'automation_cert' -> Klik tab 'SSH Keys' -> Klik '+'
#    Paste isi dari file id_rsa.pub server Anda di kolom 'Public Key'.
# 3. Ubah baris scp/ssh di atas menjadi (ganti /path/to/your/private_key dengan path ke id_rsa Anda):
#    scp -P "$MIKROTIK_SSH_PORT" -i /path/to/your/private_key -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "$FULLCHAIN_FILE" "$MIKROTIK_USER@$MIKROTIK_IP:fullchain.pem"
#    ssh -p "$MIKROTIK_SSH_PORT" -i /path/to/your/private_key -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "$MIKROTIK_USER@$MIKROTIK_IP" "/system script run import_hotspot_cert"
```
Pastikan skrip ini memiliki izin eksekusi (chmod +x push_cert_to_mikrotik.sh).

#####################################

Integrasikan dengan Cron Job/Certbot Hook:

Anda bisa menjalankan skrip ini secara manual, atau lebih baik lagi, otomatiskan dengan menambahkan ke cron atau sebagai deploy hook di Certbot.
--deploy-hook sudah di atur di automatic_certbot_file.sh, dan harus di sesuaikan untuk menjalankan otomatis file push_cert_to_mikrotik.sh jika ingin keberlangsungan yang baik

Contoh entry cron (akan dijalankan setiap hari, Certbot akan cek apakah perlu perbarui):
0 3 1 * * /opt/scripts/automatic_certbot_file.sh (jalan jam 3 pagi setiap hari dan setiap tanggal 1 di tiap bulan)
