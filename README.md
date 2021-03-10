![1](gambar/wireguard.svg)

# WireGuard

**WireGuard** merupakan salah satu VPN (*Virtual Private Network*) sederhana namun cepat dan modern yang menggunakan *cryptography*. WireGuard adalah proyek open source terbaru yang mempercepat VPN sambil membuatnya lebih aman dari sebelumnya. Secara eksplisit protokol ini diklaim lebih baik dari protokol OpenVPN dan IPsec. Awalnya dirilis hanya untuk sistem operasi Linux tetapi sekarang kompatibel dengan banyak platform lain.

# Instalasi pada Ubuntu 18.04

### Installing WireGuard
Untuk menginstall jalankan printah:
```
$ sudo apt-get update
$ sudo apt-get install wireguard
```
### Set up WireGuard pada Server

Untuk melakukan generate key jalankan perintah:
```
$ wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
```
File akan digenerate pada directory /etc/wireguard. Simpan privatekey dan jangan di*share* ke siapapun.

Buat file interface dengan nama apapun (disini kita gunakan ```wg0.conf```) dengan menjalankan perintah:
```
$ sudo nano /etc/wireguard/wg0.conf
```
Kemudian pada file ```wg0.conf``` buat interface seperti berikut:
```apacheconf
[Interface]
Address = 10.0.0.1/24
SaveConfig = true
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o enp0s3 -j MASQUERADE
```
Penjabaran Interface sebagai berikut
*Address* - Alamat IPv4 atau IPv6.
*SaveConfig* - di set true untuk menyimpan interface yang ada ketika file di non aktifkan.
*ListenPort* - Port yang digunakan untuk menerima connection.
*PrivateKey* - Yaitu PrivateKey yang didapatkan dari generate key yang telah dilakukan.
*PostUp* - Merupakan perintah untuk penyamaran ketika dijalankan yang mana disini menggunakan iptables. Perintah ini akan mengizinkan traffic meninggalkan server dan memberikan client koneksi internet.
*PostDOwn* - Merupakan perintah untuk menghentikan jalannya interface.

Sebelum menjalankan pastikan mengganti ```enp0s3``` dengan nama public interface yang ada pada device. Untuk mengetahuinya dapat dengan cara menjalankan:
```
$ ip -o -4 route show to default | awk '{print $5}'
```

Untuk membuat wg0.conf dan privatekey tidak bisa dilihat untuk normal user maka ganti permission mejadi 600:
```
$ sudo chmod 600 /etc/wireguard/{privatekey,wg0.conf}
```

Setelah selesai maka dapat dijalankan wg0 interfacenya dengan:
```
$ sudo wg-quick up wg0
```

Untuk melihat apakah berjalan atau tidak pada:
```
$ sudo wg show wg0
```

Untuk membuat WireGuard interface saat booting jalankan perintah:
```
$ sudo systemctl enable wg-quick@wg0
```

#### Server Networking dan FireWall

For NAT to work, we need to enable IP forwarding. Open the /etc/sysctl.conf file and add or uncomment the following line:
```
$ sudo nano /etc/sysctl.conf
```
```
net.ipv4.ip_forward=1
```
Save file:
```
$ sudo sysctl -p
```

If you are using UFW to manage your firewall you need to open UDP traffic on port 51820:
```
$ sudo ufw allow 51820/udp
```

# Referensi

1. [WireGuard](https://www.wireguard.com/) - Wireguard
2. [WireGuard VPN baru disempurnakan](https://id.wizcase.com/blog/wireguard-vpn-protokol-vpn-baru-dan-disempurnakan/) - idwizcase
3. [How To Set Up WireGuard on Ubuntu 18.04](https://linuxize.com/post/how-to-set-up-wireguard-vpn-on-ubuntu-18-04/) - Linuxize