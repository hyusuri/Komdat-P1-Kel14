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
### Generate key
Untuk melakukan generate key jalankan perintah:
```
$ wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
```


# Referensi

1. [WireGuard](https://www.wireguard.com/) - Wireguard
2. [WireGuard VPN baru disempurnakan](https://id.wizcase.com/blog/wireguard-vpn-protokol-vpn-baru-dan-disempurnakan/) - idwizcase
3. [How To Set Up WireGuard on Ubuntu 18.04](https://linuxize.com/post/how-to-set-up-wireguard-vpn-on-ubuntu-18-04/) - Linuxize