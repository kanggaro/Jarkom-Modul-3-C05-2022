# Jarkom-Modul-3-C05-2022

NRP|Nama|
-|-|
5025201098 | Ibra Abdi Ibadihi 
5025201028 | Muhammad Abror Al Qushoyyi
5025201156 | Nazhifah Elqolbyi 

### Laporan Resmi 

#### no6
Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP 10.12.3.13

menambahkan script pada file `/etc/dhcp/dhcpd.conf` pada weslist

```
host Eden {
  hardware ethernet 2a:a3:72:08:f4:fb;
  fixed-address 10.12.3.13;
}
```

#### no8
Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)

tambahkan script pada file `/etc/squid/acl.conf` pada berlint
```
	acl AVAILABLE_WORKING time MTWHF 08:00-17:00
	acl WEEKEND time SA 00:00-23:59
```
kemudian pada file `/etc/squid/squid.conf` tambahkan script berikut
```
	include /etc/squid/acl.conf
	http_port 8080
	visible_hostname Berlint
	http_access allow AVAILABLE_WORKING
	http_access allow WEEKEND
	http_access deny all
```

#### no9
Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)

tambahkan script pada file `/etc/squid/restrict-sites.acl` pada berlint
```
	loid-work.com
	franky-work.com
```

kemudian edit file `/etc/squid/squid.conf` dengan ini 
```
	http_port 8080
	visible_hostname Berlint

	#http_access allow AVAILABLE_WORKING
	http_access allow WEEKEND
	
	acl BLACKLISTS dstdomain "/etc/squid/restrict-sites.acl"
	http_access allow BLACKLISTS AVAILABLE_WORKING
	http_access deny all AVAILABLE_WORKING	
```

#### no10
Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)

aktifkan proxy pada client dengan `export http_proxy="http://10.12.2.3:8080"` kemudian `env | grep -i proxy`

selanjutnya pada file `/etc/squid/squid.conf` ubah dengan berikut
```
nano /etc/squid/squid.conf
http_port 8080
visible_hostname Berlint
```


#### n011
Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)

untuk membatasi bandhwitch, tambahkan file `/etc/squid/acl-bandwidth.conf` lalu tambahkan
```
	include /etc/squid/acl.conf

	delay_pools 1
	delay_class 1 1
	delay_access 1 allow all
	delay_parameters 1 128000/128000 16000/64000
```

kemudian pada `/etc/squid/squid.conf` ubah dengan berikut
```
	include /etc/squid/acl-bandwidth.conf
	http_port 8080
	visible_hostname Berlint
	http_access allow all
```
lakukan pengetesan bandwitch pada klient Garden dengan melakukan install 
```apt-install speedtest-cli```
kemudian 
```export PYTHONHTTPSVERIFY=0```
lalu jalankan speedtest pada Garden sebagai client proxy

#### no12
Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur

untuk pembatasan bandwitch hanya pada hari libur maka ubah file `/etc/squid/acl-bandwidth.conf` dengan berikut
```
	include /etc/squid/acl.conf

	delay_pools 1
	delay_class 1 1
	delay_access 1 allow all WEEKEND
	delay_access 1 deny all
	delay_parameters 1 128000/128000 16000/64000
```

lakukan pengetesan bandwitch pada klient Garden dengan melakukan install 
```apt-install speedtest-cli```
kemudian 
```export PYTHONHTTPSVERIFY=0```
lalu jalankan speedtest pada Garden sebagai client proxy


