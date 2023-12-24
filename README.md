# Fedora-DHCP-Server

Bu yazımızda Fedora Server da nasıl DHCP Server kuracağız onları göstereceğim.

Elinizde hazır Fedora Server kullanıyorsunuz diye varsayıyorum.

Terminale veya Konsola aşağıda ki komutları yazalım:

```
[root@fedoraserver ~]# dnf install sqlite
Last metadata expiration check: 2:00:26 ago on Sun 24 Dec 2023 11:56:58 AM +03.
Package sqlite-3.42.0-7.fc39.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!

```

```
[root@fedoraserver ~]# dnf install dhcp-server
Last metadata expiration check: 2:02:22 ago on Sun 24 Dec 2023 11:56:58 AM +03.
Package dhcp-server-12:4.4.3-9.P1.fc39.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```
