# KIV/PSI Úkol 4

## Topologie sítě

![Topologie sítě](scheme.png)

## Nastavení router R1

Vstup do konfiguračního rozhraní:

```
configure terminal
```

Nastavení interface připojeného do NATu (G 0/0)

```
interface gigabitEthernet 0/0
ip address dhcp
ip nat outside
no shutdown
exit
```

Nastavení interface připojeného do R2 (G 1/0):

```
interface gigabitEthernet 1/0
ip address 192.168.123.253 255.255.255.252
ip nat inside
no shutdown
exit
```

Nastavení směrování do sítě 192.168.123.0/24 - přes G 1/0:

```
ip route 192.168.123.0 255.255.255.0 gigabitEthernet 1/0
```

Dokončení konfigurace NAT:

```
access-list 1 permit 192.168.123.0 0.0.0.255
ip nat inside source list 1 interface gigabitEthernet 0/0 overload
```

Nastavení DNS (Cloudflare - 1.1.1.1)

```
ip domain-lookup
ip name-server 1.1.1.1
```

## Nastavení router R2

Nastavení interface vedoucího do R1

```
interface gigabitEthernet 0/0
ip address 192.168.123.254 255.255.255.252
no shutdown
exit
```

Nastavení interface vedoucího do switche (LAN):

```
interface gigabitEthernet 1/0
ip address 192.168.123.1 255.255.255.224
no shutdown
exit
```

Nastavení směrování veškerého provozu na R1 (default gateway):

```
ip route 0.0.0.0 0.0.0.0 192.168.123.253
```

Nastavení DNS (Cloudflare - 1.1.1.1)

```
ip domain-lookup
ip name-server 1.1.1.1
```

Nastavení DHCP. Musíme zajistit, že nebude přiřazena IP adresa 192.168.123.1 (tu má již přiřazený R2):

```
ip dhcp excluded-address 192.168.123.1
ip dhcp pool LAN
network 192.168.123.0 255.255.255.224
default-router 192.168.123.1
dns-server 1.1.1.1
exit
```

## Nastavení PC1 a PC2, testování

Po nastavení IP adresy počítačů do režimu DHCP by jim měla být přiřazena IP adresa z rozsahu 192.168.123.2 - 192.168.123.30. Do této sítě je tak teoreticky možné připojit až 29 počítačů, což splňuje zadaný požadavek na 20 počítačů.

U jednoho počítače mi ani po chvíli nedošlo k přiřazení IP adresy, pomhl restart.