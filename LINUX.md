# Nonaktifkan power saving mode pada sound di Pop!_OS 22.04 dan mengatasi masalah suara bzztt yang muncul dari speaker
Langkah-langkah untuk menonaktifkan Power Saving Mode untuk audio:

1.  Buka Terminal.
    Masukkan perintah berikut untuk mengedit file konfigurasi audio:

  `sudo nano /etc/modprobe.d/alsa-base.conf`

2.  Di dalam file tersebut, tambahkan baris berikut di bagian akhir file:

  `options snd_hda_intel power_save=0 power_save_controller=0`

3.  Simpan perubahan dengan menekan CTRL + O, lalu tekan Enter untuk mengonfirmasi nama file.
    Keluar dari editor dengan menekan CTRL + X.
    Setelah itu, reboot sistem Anda:

  `sudo reboot`

---

# Script repairing mouse freezing after sleep / suspend in linux

1. open terminal, then run this:
   `sudo nano /lib/systemd/system-sleep/fix-mouse.sh`

3. write this and save:
```
#!/bin/bash
if [ "$1" = "post" ]; then
    modprobe -r usbhid
    modprobe usbhid
fi
```

4. allow executing script
   `sudo chmod +x /lib/systemd/system-sleep/fix-mouse.sh`

---

# Fix Grub in linux for read windows OS

1. open terminal, then run this:
   `sudo apt install os-prober`

2. if done, edit this file:
   `sudo nano /etc/default/grub`

3. then write this:
    ```
    GRUB_DISABLE_OS_PROBER=false
    ```
---

# Installing Steam for Linux on an NTFS Partition

Commonly, drives are mounted automatically using /etc/fstab
. In order to install Steam for Linux onto an NTFS partition, it must be explicitly mounted using the ntfs-3g
driver. The following information lists the steps for properly modifying your /etc/fstab
file:

From a command-line, determine the UUID of the drive containing the NTFS partition:

```sudo blkid```


Add a line to your `/etc/fstab`
file that mounts that drive with `ntfs-3g`:


```UUID=yourUUID /data ntfs-3g defaults,locale=en_US.utf8 0 0```


Save your changes and reboot your machine.
