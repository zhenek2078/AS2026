# Полная настройка WireGuard туннелей

Ставим на все роутеры wireguard:

```
apt install wireguard -y
```

## Генерация ключей

### На DC-RTR-1:

Для DC-RTR-1 <-> MSK:

```
wg genkey | tee dc1_msk_priv | wg pubkey > dc1_msk_pub
```

Для DC-RTR-1 <-> YEKT:

```
wg genkey | tee dc1_yekt_priv | wg pubkey > dc1_yekt_pub
```

### На DC-RTR-2:

Для DC-RTR-2 <-> MSK:

```
wg genkey | tee dc2_msk_priv | wg pubkey > dc2_msk_pub
```

Для DC-RTR-2 <-> YEKT:

```
wg genkey | tee dc2_yekt_priv | wg pubkey > dc2_yekt_pub
```

### На MSK-RTR:

Для MSK <-> DC-RTR-1:

```
wg genkey | tee msk_dc1_priv | wg pubkey > msk_dc1_pub
```

Для MSK <-> DC-RTR-2:

```
wg genkey | tee msk_dc2_priv | wg pubkey > msk_dc2_pub
```

### На YEKT-RTR:

Для YEKT <-> DC-RTR-1:

```
wg genkey | tee yrkt_dc1_priv | wg pubkey > yrkt_dc1_pub
```

Для YEKT <-> DC-RTR-2:

```
wg genkey | tee yekt_dc2_priv | wg pubkey > yekt_dc2_pub
```

## Настройка туннельных интерфейсов

### На DC-RTR-1:

Для DC-RTR-1 <-> MSK:

```
nano /etc/wireguard/wg-msk.conf
```

```
[Interface]
Address = 10.7.7.1/30
PrivateKey = # Содержимое из dc1_msk_priv
ListenPort = 51820
Table = off

[Peer]
PublicKey = # Содержимое из msk_dc1_pub
Endpoint = 188.121.90.2:51820
AllowedIPs = 10.7.7.2/32, 224.0.0.0/24, 192.168.1.0/24
PersistentKeepAlive = 25
```

Для DC-RTR-1 <-> YEKT:

```
nano /etc/wireguard/wg-yekt.conf
```

```
[Interface]
Address = 10.6.6.1/30
PrivateKey = # Содержимое из dc1_yekt_priv
ListenPort = 51821
Table = off

[Peer]
PublicKey = # Содержимое из yekt_dc1_pub
Endpoint = 88.8.8.27:51820
AllowedIPs = 10.6.6.2/32, 224.0.0.0/24, 192.168.2.0/24
PersistentKeepalive = 25
```

Запускаем туннели:

```
wg-quick up wg-msk && systemctl enable wg-quick@wg-msk
```

```
wg-quick up wg-yekt && systemctl enable wg-quick@wg-yekt
```

### На DC-RTR-2:

Для DC-RTR-2 <-> MSK:

```
nano /etc/wireguard/wg-msk.conf
```

```
[Interface]
Address = 10.5.5.1/30
PrivateKey = # Содержимое из dc2_msk_priv
ListenPort = 51820
Table = off

[Peer]
PublicKey = # Содержимое из msk_dc2_pub
Endpoint = 188.121.90.2:51821
AllowedIPs = 10.5.5.2/32, 224.0.0.0/24, 192.168.1.0/24
PersistentKeepalive = 25
```

Для DC-RTR-2 <-> YEKT:

```
nano /etc/wireguard/wg-yekt.conf
```

```
[Interface]
Address = 10.8.8.1/30
PrivateKey = # Содержимое из dc2_yekt_priv
ListenPort = 51821
Table = off

[Peer]
PublicKey = # Содержимое из yekt_dc2_pub
Endpoint = 88.8.8.27:51821
AllowedIPs = 10.8.8.2/32, 224.0.0.0/24, 192.168.2.0/24
PersistentKeepalive = 25
```

Запускаем туннели:

```
wg-quick up wg-msk && systemctl enable wg-quick@wg-msk
```

```
wg-quick up wg-yekt && systemctl enable wg-quick@wg-yekt
```

### На MSK-RTR:

Для MSK-RTR <-> DC-RTR-1:

```
nano /etc/wireguard/wg-dc1.conf
```

```
[Interface]
Address = 10.7.7.2/30
PrivateKey = # Содержимое из msk_dc1_priv
ListenPort = 51820
Table = off

[Peer]
PublicKey = # Содержимое из dc1_msk_pub
Endpoint = 200.100.100.20:51820
AllowedIPs = 10.7.7.1/32, 224.0.0.0/24, 10.15.10.0/24, 192.168.2.0/24
PersistentKeepalive = 25
```

Для MSK-RTR <-> DC-RTR-2:

```
nano /etc/wireguard/wg-dc2.conf
```

```
[Interface]
Address = 10.5.5.2/30
PrivateKey = # Содержимое из msk_dc2_priv
ListenPort = 51821
Table = off

[Peer]
PublicKey = # Содержимое из dc2_msk_pub
Endpoint = 100.200.100.20:51820
AllowedIPs = 10.5.5.1/32, 224.0.0.0/24, 10.15.10.0/24, 192.168.2.0/24
PersistentKeepalive = 25
```

Запускаем туннели:

```
wg-quick up wg-dc1 && systemctl enable wg-quick@wg-dc1
```

```
wg-quick up wg-dc2 && systemctl enable wg-quick@wg-dc2
```

### На YEKT-RTR:

Для YEKT-RTR <-> DC-RTR-1:

```
nano /etc/wireguard/wg-dc1.conf
```

```
[Interface]
Address = 10.6.6.2/30
PrivateKey = # Содержимое из  yekt_dc1_priv
ListenPort = 51820
Table = off

[Peer]
PublicKey = # Содержимое из dc1_yekt_pub
Endpoint = 200.100.100.20:51821
AllowedIPs = 10.6.6.1/32, 224.0.0.0/24, 10.15.10.0/24, 192.168.1.0/24
PersistentKeepalive = 25
```

Для YEKT-RTR <-> DC-RTR-2:

```
nano /etc/wireguard/wg-dc2.conf
```

```
[Interface]
Address = 10.8.8.2/30
PrivateKey = # Содержимое из yekt_dc2_priv
ListenPort = 51821
Table = off

[Peer]
PublicKey = # Содержимое из dc2_yekt_pub
Endpoint = 100.200.100.20:51821
AllowedIPs = 10.8.8.1/32, 224.0.0.0/24, 10.15.10.0/24, 192.168.1.0/24
PersistentKeepalive = 25
```

Запускаем туннели:

```
wg-quick up wg-dc1 && systemctl enable wg-quick@wg-dc1
```

```
wg-quick up wg-dc2 && systemctl enable wg-quick@wg-dc2
```
