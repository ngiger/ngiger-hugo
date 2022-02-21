---
title: "Dnsmasq"
date: 2022-02-20T19:32:48+01:00
draft: false
---
# DNSMASQ as DHCP server in my local network

Changes to the minimal setup of the NixOS setup

* Open firewall
  * DNS port 53 # DNS
  * DHCP port 67
* Add list of mac-addr -> ip
* Defined gateway
* Defined nameservers

The NixOS snippet is

```Nix
{ pkgs, config, ...}:
{
# leases are found under /var/lib/dnsmasq/dnsmasq.leases
  environment.systemPackages = [ pkgs.dnsmasq pkgs.bind.dnsutils ]; 
  networking.firewall.allowedUDPPorts = [
    67 # DHCP
    53 # DNS
  ];
  networking.nameservers = [ "127.0.0.1" # we must specify this first for dnsmasq
                      "62.2.17.60" # UPC
                      "8.8.8.8" # google
                    ];
  services.dnsmasq.enable = true;
  services.dnsmasq.alwaysKeepRunning = true;
  services.dnsmasq.servers = ["8.8.8.8" "62.2.17.60" "62.2.24.162" ]; # google and cablecom.net
  services.dnsmasq.extraConfig = ''
dhcp-option=3,${cfg.upc-router.ipv4} # UPC router
dhcp-range=192.168.0.200,192.168.0.250,12h
dhcp-authoritative
# listen (aka answer DHCP request on) IP of static ethernet, IP of WLAN, local
listen-address=192.168.0.5,192.168.0.5.244,127.0.0.1
# Give my Raspberry PI alwas the IP .10
dhcp-host=B8:27:EB:19:17:27,192.168.0.10,my_PI3,3h
'';
}
```
