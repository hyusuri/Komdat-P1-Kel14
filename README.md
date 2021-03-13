![1](gambar/wireguard.svg)


[WireGuard](#WireGuard) | [Instalasi](#Instalasi) | [Set up WireGuard pada Server](#setup) | [Set up WireGuard pada Client Linxu](#setupclient) | [Menghubungkan Client dengan VPN](#menghubungkan) | [Referensi](#referensi)
:---:|:---:|:---:|:---:|:---:|:---:


# WireGuard
[`kembali ke atas`](#)


**WireGuard** merupakan salah satu VPN (*Virtual Private Network*) sederhana namun cepat dan modern yang menggunakan *cryptography*. WireGuard adalah proyek open source terbaru yang mempercepat VPN sambil membuatnya lebih aman dari sebelumnya. Secara eksplisit protokol ini diklaim lebih baik dari protokol OpenVPN dan IPsec. Awalnya dirilis hanya untuk sistem operasi Linux tetapi sekarang kompatibel dengan banyak platform lain.

# Instalasi pada Ubuntu 18.04
[`kembali ke atas`](#)

### Installing WireGuard
Untuk menginstall jalankan printah:
```
$ sudo apt-get update
$ sudo apt-get install wireguard
```
# Set up WireGuard pada Server
[`kembali ke atas`](#)

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

### Server Networking dan FireWall

untuk enable IP forwarding buka file /etc/sysctl.conf
```
$ sudo nano /etc/sysctl.conf
```
Tambahkan atau hilangkan komen yang ada pada line berikut:
```
net.ipv4.ip_forward=1
```
Save file dengan:
```
$ sudo sysctl -p
```

Jika menggunakan UFW untuk mengatur firewall maka perlu untuk membuka UDP pada port 51820 dengan:
```
$ sudo ufw allow 51820/udp
```

# Set up WireGuard pada Client Linux
[`kembali ke atas`](#)

Pertama install WireGuard sama seperti instalasi pada server. Kemudian lakukan generate public dan private key dengan menjalankan:
```
$ wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
```
setelah melakukan generate key, selanjutnya membuat file ```wg0.conf``` dengan:
```
$ sudo nano /etc/wireguard/wg0.conf
```
kemudian isi wg0.conf dengan:
```
[Interface]
PrivateKey = CLIENT_PRIVATE_KEY
Address = 10.0.0.2/24


[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = SERVER_IP_ADDRESS:51820
AllowedIPs = 0.0.0.0/0
```
**Untuk interface**:

*PrivateKey* - Privatekey client yang sudah di generate sebelumnya.

*Address* - alamat untuk wg0 interface.

**Untuk Peer**

*PublicKey* - Publickey milik server yang di generate pada server.

*Endpoint* - IP address dari server yang akan disambungkan dan diikuti oleh port yang listening pada server.

*AllowedIPs* - alamat ip yang mana trafic masuk yang diizinkan dan trafic keluar yang diarahkan.


#### Menambahkan Client Peer pada Server

untuk menambahkan client peer jalankan:
```
$ sudo wg set wg0 peer CLIENT_PUBLIC_KEY allowed-ips 10.0.0.2
```

```CLIENT_PUBLIC_KEY``` adalah publickey yang di generate pada komputer client.

# Menghubungkan Linux Client dengan VPN WireGuard
[`kembali ke atas`](#)

Pastikan bahwa WireGuard pada server sudah jalan kemudian pada client jalankan perintah:
```
$ sudo wg-quick up wg0
```
Sekarang client sudah terrhubung dengan server. Untuk mengecek dapat menjalankan perintah:
```
$ sudo wg
```
Kemudian untuk mengecek IP dapat menggunakan google dengan mencari ```what is my ip```

Untuk menghentikan sambungan VPN dapat dengan menjalankan perintah:
```
$ sudo wg-quick down wg0
```


# Referensi
[`kembali ke atas`](#)

1. [WireGuard](https://www.wireguard.com/) - Wireguard
2. [WireGuard VPN baru disempurnakan](https://id.wizcase.com/blog/wireguard-vpn-protokol-vpn-baru-dan-disempurnakan/) - idwizcase
3. [How To Set Up WireGuard on Ubuntu 18.04](https://linuxize.com/post/how-to-set-up-wireguard-vpn-on-ubuntu-18-04/) - Linuxize