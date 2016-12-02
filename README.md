# Ansible Role: Shorewall

## Description

Ansible role which installs and configures shorewall and shorewall6.

## Installation

```
$ ansible-galaxy install sbaerlocher.shorewall
```

## Requirements

## Role Variables

```yaml
shorewall_interfaces:
  - { zone: net, interface: eth0, options: "dhcp,tcpflags,logmartians,nosmurfs,sourceroute=0" }

shorewall_policies:
  - { source: "$FW", dest: all, policy: ACCEPT }
  - { source: net, dest: all, policy: REJECT }
  - { source: all, dest: all, policy: REJECT, log_level: info }

shorewall_rules:
  - section: NEW
    rules:
    - { action: "Invalid(DROP)", source: net, dest: "$FW", proto: tcp }
    - { action: ACCEPT, source: net, dest: "$FW", proto: tcp, dest_port: ssh }
    - { action: ACCEPT, source: net, dest: "$FW", proto: icmp, dest_port: echo-request }

shorewall_zones:
  - { zone: fw, type: firewall }
  - { zone: net, type: ipv4 }
```
  
## Dependencies

## Example Playbook

```yml
- hosts: all
  roles:
     - sbaerlocher.shorewall
```

## Changelog

## Author

* [Simon Bärlocher](https://sbaerlocher.ch)
* Farhad Shahbazi
* Sascha Biberhofer
 
## License

This project is under the MIT License. See the [LICENSE](https://sbaerlo.ch/licence) file for the full license text.

## Copyright

(c) 2016, Simon Bärlocher