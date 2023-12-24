# Fedora DCHP Server Kurulumu

Bu yazıda, Fedora Server üzerinde basit bir şekilde DHCP (Dynamic Host Configuration Protocol) sunucusu kurmayı göstereceğim. DHCP sunucusu, ağdaki cihazlara otomatik IP adresleri atamak ve ağ yapılandırmasını kolaylaştırmak için kullanılır.

## Kurulum

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


## LOG

Aşağıda ki dosya yoluna giderek  IP kiralama loglarına bakabilirsiniz:

```
nano /var/lib/dhcpd/dhcpd.leases
```
```
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.4.3-P1

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

server-duid "\000\001\000\001-\032\306\004\000\014)\302W ";

lease 172.16.1.150 {
  starts 0 2023/12/24 10:53:57;
  ends 0 2023/12/24 11:03:57;
  cltt 0 2023/12/24 10:53:57;
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 00:0c:29:b7:57:f7;
  uid "\001\000\014)\267W\367";
  set vendor-class-identifier = "MSFT 5.0";
  client-hostname "WIN-SKI3OII7L7G";
}
```

Şimdi logları bir dosyaya yazdıralım ilk önce `nano /etc/dhcp/dhcpd.conf` yoluna giderek aşağıdaki komutu yazınız:

```
# event logs
log-facility local7; # 7 sayısı Network loglarını temsil eder.
```

İşlemimize devam etmek için **rsyslogu** indireceğiz

```
[root@fedoraserver etc]# sudo dnf install rsyslog
Last metadata expiration check: 0:19:58 ago on Sun 24 Dec 2023 02:23:35 PM +03.
Package rsyslog-8.2310.0-1.fc39.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

Şimdi `nano /etc/rsyslog.conf` yoluna giderek gerekli yapılandırmaları yapalım. Dosyayı açtığımızda `#Save boot messages also to boot.log` kısmını bulup aşağıdaki komutu yazıyoruz

```
# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log # bunu silmenize gerek yok
local7.*                                                /var/log/dhcp.log # Loglanacak dosya yolu
```

Bunları yaptıktan sonra servisi yeniden başlatacağız:

```
systemctl restart rsyslog.service
systemctl restart dhcpd
```

Gerekli işlemleri yaptıktan sonra Windows Sanal Makinanızın internetini kapatıp açabilirsiniz. `nano /var/log/dhcp.log` dosya yoluna gittiğinizde aşağıda ki logları göreceksiniz

### Loglar
```
Dec 24 14:39:00 fedoraserver dhcpd[1445]: Internet Systems Consortium DHCP Server 4.4.3-P1
Dec 24 14:39:00 fedoraserver dhcpd[1445]: Copyright 2004-2022 Internet Systems Consortium.
Dec 24 14:39:00 fedoraserver dhcpd[1445]: All rights reserved.
Dec 24 14:39:00 fedoraserver dhcpd[1445]: For info, please visit https://www.isc.org/software/dhcp/
Dec 24 14:39:00 fedoraserver dhcpd[1445]: Source compiled to use binary-leases
Dec 24 14:39:00 fedoraserver dhcpd[1445]: Wrote 1 leases to leases file.
Dec 24 14:39:00 fedoraserver dhcpd[1445]: Listening on LPF/ens160/00:0c:29:c2:57:20/172.16.1.0/24
Dec 24 14:39:00 fedoraserver dhcpd[1445]: Sending on   LPF/ens160/00:0c:29:c2:57:20/172.16.1.0/24
Dec 24 14:39:00 fedoraserver dhcpd[1445]: Sending on   Socket/fallback/fallback-net
Dec 24 14:39:00 fedoraserver dhcpd[1445]: Server starting service.
Dec 24 14:39:29 fedoraserver dhcpd[1445]: DHCPDISCOVER from 00:0c:29:b7:57:f7 via ens160
Dec 24 14:39:30 fedoraserver dhcpd[1445]: DHCPOFFER on 172.16.1.150 to 00:0c:29:b7:57:f7 (DESKTOP-VN7AMBD) via ens160
Dec 24 14:39:30 fedoraserver dhcpd[1445]: DHCPREQUEST for 172.16.1.150 (172.16.1.20) from 00:0c:29:b7:57:f7 (DESKTOP-VN7AMBD) via ens160
Dec 24 14:39:30 fedoraserver dhcpd[1445]: DHCPACK on 172.16.1.150 to 00:0c:29:b7:57:f7 (DESKTOP-VN7AMBD) via ens160
Dec 24 14:44:30 fedoraserver dhcpd[1445]: DHCPREQUEST for 172.16.1.150 from 00:0c:29:b7:57:f7 (DESKTOP-VN7AMBD) via ens160
Dec 24 14:44:30 fedoraserver dhcpd[1445]: DHCPACK on 172.16.1.150 to 00:0c:29:b7:57:f7 (DESKTOP-VN7AMBD) via ens160
```


## IP Rezervasyon

Bu işlemi yapmamızdaki sebeplerden birisi şu; Örneğin DHCP Serverdan tarafından bir bilgisayara 172.16.1.155 IP adresi atanmış ve Firewallda ise bu IP adresinin facebook.com sitesine gitmek yasak diye varsayalım. Bu IP adresi başka bir bilgisayara atandığında artık o bilgisayar facebook.com web sitesine gidemeyecek. Aynısı mobil cihazlar içinde geçerlidir.

Bu işlemi yapmak basittir. İlk önce `nano /etc/dhcp/dhcpd.conf` dosya yoluna gidip aşağıda ki kodu yazınız:

```
host win {
    hardware ethernet 00:0C:29:B7:57:F7; # Cihazın Mac Adresi
    fixed-address 172.16.1.155; # Rezervasyon yapılacak IP Adresi
}
```

Bunu yaptıktan sonra dhcp servisini yeniden başlatalım:

```
systemctl restart dhcpd.service
```

Bunu yaptıktan sonra Windows Makinamızda terminale aşağıda ki komutları yazarak Rezervasyon yaptığımız IP adresini atama işlemini kontrol edebilirsiniz.

```
ipconfig /release # Mevcut IP adresini bırakmak için kullanılır.
ipconfig /renew # Yeni bir IP adresi talep etmek ve almak için kullanılır.
```

![image](https://github.com/ugurcomptech/Fedora-DHCP-Server/assets/133202238/3963989f-b3d5-4b73-906d-cfacca2849d9)


![image](https://github.com/ugurcomptech/Fedora-DHCP-Server/assets/133202238/cd1371bb-0d36-45c6-82cf-1ac2c6086f1d)







