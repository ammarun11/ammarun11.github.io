## Prasyarat

-   OpenStack User
-   Horizon
-   Instance RHEL/CentOS/Rocky

## Panduan

1.  Buka console instance pada horizon
    
2.  Masuk ke boot option dengan cara interupsi proses booting
    
    -   Reboot instance dan tekan tombol `panah` arah apapun di keyboard pada saat proses reboot
3.  Saat masuk boot option pilih kernel dan tekan tombol `e` pada keyboard
    
    -   Cari baris dengan awalan `linux /boot/vmlinuz-xxxxxxxx`
    -   Tambahkan baris berikut `rd.break` pada baris yang sama dengan `linux /boot/vmlinuz-xxxxxxxx`
    -   Hapus kata berikut `console=ttyS0`
    -   Lalu tekan `Ctrl + x`
4.  Reset password root menggunakan perintah `passwd`
    
    -   Jika sudah masuk maka tampilan terminal adalah sebagai berikut
    
    -   Mount ulang /sysroot dengan menggunakan perintah berikut
    
    ```
    mount -o remount,rw /sysroot
    ```
    
    -   Jadikan `/sysroot` sebagai root directory
    
    -   Lalu ganti password menggunakan perintah `passwd`
    
    -   Relabel filesystem SELinux saat reboot
5.  Reboot sistem menggunakan perintah berikut (2 kali exit)
    

```
# Keluar dari chroot
exit

# Keluar untuk melanjutkan ke boot normal
exit
```

## Referensi

-   [Reset OpenStack Instance Password](https://jpenatech.wordpress.com/2017/04/20/successfully-resetting-the-root-password-of-a-centos-7-vm-in-openstack/)
-   [Reset Root Password Red Hat](https://linuxconfig.org/redhat-8-recover-root-password)
-   [What is autorelabel](https://serverfault.com/questions/432531/what-does-the-autorelabel-file-do-in-linux)
