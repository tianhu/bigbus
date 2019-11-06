![picture](cisco-anyconnect-icon.png)
This guide can help you setup a OpenConnect VPN server (OCServ) on Ubuntu 1804.

# Chapter 1 Set up a VPN server

## Install OCServ

```
sudo apt install ocserv
```

## Install Let’s Encrypt client

```
sudo add-apt-repository ppa:certbot/certbot
sudo apt install certbot
```

## Obtain certificate

```
sudo certbot certonly --standalone --preferred-challenges http --agree-tos --email <your-email> -d <your-vpn-domain>
```

## Auto-Renew Let’s Encrypt Certificate

```
sudo crontab -e
```

> Add the following line at the end of the file to run the Cron job daily.
>> `@daily certbot renew --quiet && systemctl restart ocserv`

## Configure OpenConnect VPN server

```
sudo vi /etc/ocserv/ocserv.conf
```

> * tcp-port = 443
> * #udp-port = 443
> * server-cert = /etc/letsencrypt/live/<your-vpn-domain>/fullchain.pem
> * server-key = /etc/letsencrypt/live/<your-vpn-domain>/privkey.pem
> * max-clients = 0
> * max-same-clients = 0
> * try-mtu-discovery = true
> * default-domain = <your-vpn-domain>
> * ipv4-network = 192.168.50.0
> * ipv4-netmask = 255.255.255.0
> * tunnel-all-dns = true
> * dns = 8.8.8.8
> * dns = 8.8.4.4
> * #route = 10.10.10.0/255.255.255.0
> * #route = 192.168.0.0/255.255.0.0
> * #route = fef4:db8:1000:1001::/64
> * #route = default
> * #no-route = 192.168.5.0/255.255.255.0

```
sudo systemctl restart ocserv
```

## Add VPN user

```
sudo useradd <username>
sudo passwd <username>
```

## Enable IP forwarding

```
sudo vi /etc/sysctl.conf
```

> `net.ipv4.ip_forward = 1`

```
sudo sysctl -p
sudo iptables -t nat -A POSTROUTING -s 192.168.50.0/24 -o eth0 -j MASQUERADE
sudo apt install iptables-persistent
```


# Chapter 2 Set up Clients

## Linux client
```
sudo apt install openconnect network-manager-openconnect network-manager-openconnect-gnome
```
> Use network manager GUI or run below command line.
>> `sudo openconnect -b <your-vpn-domain>`
>> To stop the connection, run:
>> `sudo pkill openconnect`

## Windows, Mac

You have 3 options below. I strongly recommend choosing option 3, openconnect-gui, an opensource alternative to Cisco AnyConnect.

* Use the system built-in Cisco VPN provider.
* Install Cisco AnyConnect.
* Install openconnect-gui from https://github.com/openconnect/openconnect-gui/releases

## Android, iOS

Install Cisco AnyConnect from their App Store.
