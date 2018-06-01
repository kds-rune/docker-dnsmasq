## Supported tags ##

* latest

## Quick Reference ##

Maintained by: <rune.ronneseth@kred.no>

Versions (latest):
- Alpine: v3.7
- Dnsmasq: v2.78

## Description ##

This container runs [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html), and adds ipxe-files at /var/lib/tftpboot

## Usage ##

Note:
* -d means run detached/daemon/background
* --rm means remove the container once it stops (e.g. when you exit)
* --network=host lets you use the hosts network stack (communicate outside the docker network)
* --cap-add=NET_ADMI gives access to modiy host network interface (i.e. for dhcp)

#### Start docker container ####

Modify as suits your need:
- subnet: 172.16.8.0/22
- dnsmasq/host-ip: 172.16.10.1
- dhcp-range: 172.16.10.100 > 172.16.10.200
- gateway: 172.16.8.100
```
#!/bin/bash
sudo docker run -d --rm --cap-add=NET_ADMIN --network="host" runeronneseth/dnsmasq:latest \
  --no-daemon \
  --no-resolv \
  --log-queries \
  --log-dhcp \
  --port=5353 \
  --enable-tftp \
  --tftp-root=/var/lib/tftpboot \
  --dhcp-range=172.16.10.100,172.16.10.200,10m \
  --dhcp-option=3,172.16.8.100 \
  --strict-order \
  --server=127.0.0.1 \
  --server=208.67.222.222 \
  --server=208.67.220.220 \
  --server=8.8.8.8 \
  --server=8.8.4.4 \
  --dhcp-match=set:bios,option:client-arch,0 \
  --dhcp-boot=tag:bios,undionly.kpxe \
  --dhcp-match=set:efi32,option:client-arch,6 \
  --dhcp-boot=tag:efi32,ipxe.efi \
  --dhcp-match=set:efibc,option:client-arch,7 \
  --dhcp-boot=tag:efibc,ipxe.efi \
  --dhcp-match=set:efi64,option:client-arch,9 \
  --dhcp-boot=tag:efi64,ipxe.efi \
  --dhcp-userclass=set:ipxe,iPXE \
  --dhcp-boot=tag:ipxe,http://boot.ipxe.org/demo/boot.php
```

Running as dhcp-proxy (..,proxy,..), without dns (--port=0)
```
#!/bin/bash
docker run -d --rm --cap-add=NET_ADMIN --network="host" runeronneseth/dnsmasq:latest \
  --port=0 \
  --no-daemon \
  --log-dhcp \
  --log-queries \
  --enable-tftp \
  --tftp-root=/var/lib/tftpboot \
  --dhcp-range=172.16.8.0,proxy,255.255.252.0 \
  --dhcp-match=set:ipxe,175 \
  --pxe-service=tag:#ipxe,x86PC,"Chainload ipxe",undionly.kpxe \
  --pxe-service=tag:#ipxe,BC_EFI,"Chainload ipxe",ipxe.efi \
  --pxe-service=tag:ipxe,x86PC,"Loading ipxe-menu",http://boot.ipxe.org/demo/boot.php \
  --pxe-service=tag:ipxe,BC_EFI,"Loading ipxe-menu",http://boot.ipxe.org/demo/boot.php
```

## Disclaimer ##

..
