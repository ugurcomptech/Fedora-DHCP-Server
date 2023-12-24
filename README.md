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


Gerekli paketleri indirdik şimdi bunları yapılandıralım. `nano /etc/dhcp/dhcpd.conf` dosya yoluna gidelim. Tavsiyem VMWare üzerinde deniyorsanız bu yapılandırmaları **NAT** Networkünde devam etmenizdir. VMWare üzerinden VM Network Editör sekmesine gidip NAT seçeneğine tıkladıktan sonra gerekli yapılandırmaları kendinize göre yapabilirsiniz.

`nano /etc/dhcp/dhcpd.conf` içerisine aşağıda ki komutları yazalım

```
# DHCP sunucusu tarafından yapılan tanımlamaların yetkili (authoritative) olmasını belirtir.
authoritative;

# Kira süresi tanımlamaları
default-lease-time 600;   # Varsayılan kira süresi (saniye cinsinden)
max-lease-time 7200;      # Maksimum kira süresi (saniye cinsinden)

# Subnet tanımlaması
subnet 172.16.1.0 netmask 255.255.255.0 {
    range 172.16.1.150 172.16.1.200;   # IP adres aralığı DHCP tarafından atanabilir
    option routers 172.16.1.253;       # Gateway (ağ geçidi) IP adresi
    option domain-name-servers 1.1.1.1, 8.8.8.8;  # DNS sunucu IP adresleri
}
```

Gerekli yapılandırmaları yaptıtkan sonra DHCP servisimizi bir kontrol edelim:

```
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; preset: disabled)
    Drop-In: /usr/lib/systemd/system/service.d
             └─10-timeout-abort.conf
     Active: active (running) since Sun 2023-12-24 13:35:48 +03; 29min ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 1148 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 4617)
     Memory: 4.5M
        CPU: 18ms
     CGroup: /system.slice/dhcpd.service
             └─1148 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Dec 24 13:53:53 fedoraserver dhcpd[1148]: DHCPDISCOVER from 00:0c:29:b7:57:f7 via ens160
Dec 24 13:53:54 fedoraserver dhcpd[1148]: DHCPOFFER on 172.16.1.150 to 00:0c:29:b7:57:f7 (WIN-SKI3OII7L7G) via ens160
Dec 24 13:53:57 fedoraserver dhcpd[1148]: DHCPDISCOVER from 00:0c:29:b7:57:f7 (WIN-SKI3OII7L7G) via ens160
Dec 24 13:53:57 fedoraserver dhcpd[1148]: DHCPOFFER on 172.16.1.150 to 00:0c:29:b7:57:f7 (WIN-SKI3OII7L7G) via ens160
Dec 24 13:53:57 fedoraserver dhcpd[1148]: DHCPREQUEST for 172.16.1.150 (172.16.1.20) from 00:0c:29:b7:57:f7 (WIN-SKI3OII7L7G) via ens160
Dec 24 13:53:57 fedoraserver dhcpd[1148]: DHCPACK on 172.16.1.150 to 00:0c:29:b7:57:f7 (WIN-SKI3OII7L7G) via ens160
Dec 24 13:57:02 fedoraserver dhcpd[1148]: DHCPREQUEST for 172.16.1.150 from 00:0c:29:b7:57:f7 (WIN-SKI3OII7L7G) via ens160
Dec 24 13:57:02 fedoraserver dhcpd[1148]: DHCPACK on 172.16.1.150 to 00:0c:29:b7:57:f7 (DESKTOP-VN7AMBD) via ens160
Dec 24 14:01:32 fedoraserver dhcpd[1148]: DHCPREQUEST for 172.16.1.150 from 00:0c:29:b7:57:f7 (DESKTOP-VN7AMBD) via ens160
Dec 24 14:01:32 fedoraserver dhcpd[1148]: DHCPACK on 172.16.1.150 to 00:0c:29:b7:57:f7 (DESKTOP-VN7AMBD) via ens160
```


Eğer bir hata alırsanız servisi yeniden başlatabilirsiniz

```
systemctl restart dhcpd.service
systemctl status dhcpd.service
```

Tekrardan hata almanız durumunda aşağıda ki komutları yazarak loglara bakabilirsiniz

```
journalctl -xe | grep dhcpd
```
```
journalctl -xeu dhcpd.service
```

Şimdi tüm hataları giderdiğimize göre bunu sanal Windows bir makinada test edelim. İkisininde NAT Networkünde olduğuna dikkat edin yoksa çalışmaz.

![image](https://github.com/ugurcomptech/Fedora-DHCP-Server/assets/133202238/1af24e7c-2236-41a0-a134-181512edbf34)

Serverımız başarılı bir şekilde çalıştı.









