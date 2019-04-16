# Ansible Role: Shorewall

[![Build Status](https://img.shields.io/travis/arillso/ansible.shorewall.svg?branch=master&style=popout-square)](https://travis-ci.org/arillso/ansible.shorewall) [![license](https://img.shields.io/github/license/mashape/apistatus.svg?style=popout-square)](https://sbaerlo.ch/licence) [![Ansible Galaxy](http://img.shields.io/badge/ansible--galaxy-shorewall-blue.svg?style=popout-square)](https://galaxy.ansible.com/arillso/shorewall) [![Ansible Role](https://img.shields.io/ansible/role/d/24784.svg?style=popout-square)](https://galaxy.ansible.com/arillso/shorewall)

## Description

Ansible role which installs and configures [Shorewall](http://shorewall.org/) and Shorewall6.

## Installation

```bash
ansible-galaxy install arillso.shorewall
```

## Requirements

Ansible version 2.0 or better.

## Role Handlers

| Name                                      | Description                                |
| ----------------------------------------- | ------------------------------------------ |
| `enable shorewall`, `enable shorewall6`   | Enables and starts Shorewall / Shorewall 6 |
| `restart shorewall`, `restart shorewall6` | Restarts Shorewall / Shorewall6            |

## Role Variables

_Note:_ The Shorewall (IPv4) variables are prefixed by `shorewall_`, whereas the Shorewall6 (IPv6) variables are prefixed by `shorewall6_`.

| Variable                | Dictionary / Options                                                                                                                                                                                                                            |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| shorewall_package_state | "present", "latest", "absent".                                                                                                                                                                                                                  |
| shorewall_startup       | "1" or "0"                                                                                                                                                                                                                                      |
| shorewall_conf          | _this variable uses standard option / value pairs_                                                                                                                                                                                              |
| shorewall_interfaces    | `zone`, `interface`, `options`                                                                                                                                                                                                                  |
| shorewall_zones         | `zone`, `type`, `options`, `options_in`, `options_out`                                                                                                                                                                                          |
| shorewall_policies      | `source`, `dest`, `policy`, `log_level`, `burst_limit`, `conn_limit`                                                                                                                                                                            |
| shorewall_rules         | **sections**: `section`, **rules**: `rule`. For each **rule**: `action`, `source`, `dest`, `proto`, `dest_port`, `source_port`, `original_dest`, `rate_limit`, `user_group`, `mark`, `connlimit`, `time`, `headers`, `switch`, `helper`, `when` |
| shorewall_masq          | `interface`, `source`, `address`, `proto`, `ports`, `ipsec`, `mark`, `user`, `switch`, `original_dest`                                                                                                                                          |
| shorewall_tunnels       | `type`, `zone`, `gateway`, `gateway_zone`                                                                                                                                                                                                       |
| shorewall_hosts         | `zone`, `hosts`, `options`                                                                                                                                                                                                                      |
| shorewall_params        | `name`, `value`                                                                                                                                                                                                                                 |

### shorewall_package_state - Shorewall package state

See the Ansible [package module](http://docs.ansible.com/ansible/package_module.html) information for more details.

It allows you to control whether Shorewall and dependencies should be either installed (_"present"_), installed / upgraded to their most recent version (_"latest"_) or should be removed (_"absent"_).

### shorewall_startup - Shorewall startup behaviour

This updates the `/etc/default/shorewall` file's `startup` option to either enable (_"1"_) startup (using the `service` or `systemctl` commands) or disable it (_"0"_).

### shorewall_conf - Shorewall Configuration

Specify values for global Shorewall options in the `/etc/shorewall/shorewall.conf` file. See the Shorewall [shorewall.conf man page](http://shorewall.org/manpages/shorewall.conf.html) for more details.

Each shorewall.conf option may be written in lower-case, such as `ACCEPT_DEFAULT=none` can be written as `accept_default: "none"` in the variables.

#### Example

```yaml
shorewall_conf:
  verbosity: '1'
  log_verbosity: '2'
  logfile: '/var/log/messages'
  blacklist: '"NEW,INVALID,UNTRACKED"'
  blacklist_disposition: 'DROP'
```

### shorewall_interfaces - Interfaces

Define the interfaces on the system and optionally associate them with zones in the `/etc/shorewall/interfaces` file. See the Shorewall [interfaces man page](http://www.shorewall.net/manpages/shorewall-interfaces.html) for more details.

#### Example - Interfaces

```yaml
shorewall_interfaces:
  - {
      zone: net,
      interface: eth0,
      options: 'dhcp,tcpflags,logmartians,nosmurfs,sourceroute=0',
    }
```

### shorewall_zones - Zones

Declare Shorewall zones in the `/etc/shorewall/zones` file. See the Shorewall [zones man page](http://www.shorewall.net/manpages/shorewall-zones.html) for more details.

#### Example - Zone

```yaml
shorewall_zones:
  - { zone: fw, type: firewall }
  - { zone: net, type: ipv4 }
```

### shorewall_policies - Policies

Define high-level policies for connections between zones in the `/etc/shorewall/policies`. See the Shorewall [policy man page](http://www.shorewall.net/manpages/shorewall-policy.html) for more details.

#### Example - Policies

```yaml
shorewall_policies:
  - { source: '$FW', dest: all, policy: ACCEPT }
  - { source: net, dest: all, policy: REJECT }
  - { source: all, dest: all, policy: REJECT, log_level: info }
```

### shorewall_rules - Rules

Specify exceptions to policies, including DNAT and REDIRECT in the `/etc/shorewall/rules` file. See the Shorewall [rules man page](http://www.shorewall.net/manpages/shorewall-rules.html) for more details.

**_WARNING_**: Please be sure to include a rule for SSH on the correct port, to avoid locking Ansible - and yourself - out from the remote host.

#### Using the `when` conditional

An option specific to this role variable. and not part of Shorewall, is the `when` conditional. This allows a rule to be included only if the condition evaluates to True.

#### Examples

```yaml
shorewall_rules:
  - section: NEW
    rules:
      - { action: 'Invalid(DROP)', source: net, dest: '$FW', proto: tcp }
      - { action: ACCEPT, source: net, dest: '$FW', proto: tcp, dest_port: ssh }
      - {
          action: ACCEPT,
          source: net,
          dest: '$FW',
          proto: icmp,
          dest_port: echo-request,
        }
```

Using the `when` conditional:

```yaml
has_webserver: True

# And in a task:
#- name: Disable webserver rule
#  set_fact:
#    has_webserver: False

shorewall_rules:
  - section: NEW
    rules:
      - { action: 'Invalid(DROP)', source: net, dest: '$FW', proto: tcp }
      - { action: ACCEPT, source: net, dest: '$FW', proto: tcp, dest_port: ssh }
      - {
          action: 'HTTP(ACCEPT)',
          source: net,
          dest: '$FW',
          when: '{{ has_webserver }}',
        }
```

### shorewall_masq - Masquerade/SNAT

Define Masquerade/SNAT in the `/etc/shorewall/masq` file. See the Shorewall [masq man page](http://shorewall.org/manpages/shorewall-masq.html) for more details.

### shorewall_tunnels - Tunnels

Define VPN connections with endpoints on the firewall in the `/etc/shorewall/tunnels` file. See the Shorewall [tunnels man page](http://shorewall.net/manpages/shorewall-tunnels.html) for more details.

#### Example - Tunnels

```yaml
shorewall_tunnels:
  - { type: ipsec, zone: net, gateway: '0.0.0.0/0', gateway_zones: 'vpn1,vpn2' }
```

### shorewall_hosts - Hosts

Define multiple zones accessed through a single interface in the `/etc/shorewall/hosts` file. See the Shorewall [hosts man page](http://shorewall.org/manpages/shorewall-hosts.html) for more details.

### shorewall_params - Parameters

Assign any shell variables that you need in the `/etc/shorewall/params` file. See the Shorewall [params man page](http://shorewall.org/manpages/shorewall-params.html) for more details.

## Example Playbook

```yml
- hosts: all
  roles:
    - arillso.shorewall
```

## Changelog

### v1.0.9

- change variable from `ansible_ssh_port` to `shorewall_ssh_default`

### v1.0.8

- Enable and start service (`use: service` is set because of [https://github.com/ansible/ansible/issues/22303](https://github.com/ansible/ansible/issues/22303)) (thank [S. Veyrié](https://github.com/turb))

### v1.0.7

- Added support for shorewall SNAT configuration (thank [ErmakovDmitriy](https://github.com/ErmakovDmitriy))

### v1.0.6

- Sort shorewall.conf items to ensure idempotence between runs (thank [S. Veyrié](https://github.com/turb))

### v1.0.5

- rename role name

### v1.0.4

- add support for docker
- add tags support

### v1.0.3

- Added: The `shorewall_rules` has an added option `when` for each rule, which acts similar to Ansible's `when` statement and allows rules to be conditional.
- Added: role variable `shorewall_tunnels` for use with VPNs.
- _Changed:_ The generated `shorewall_rules` will now take into account the `?` prefix in sections (i.e. `?ESTABLISHED`), which was introduced at Shorewall version 4.6. If the Shorewall version installed is older than 4.6, this prefix will be omitted to avoid errors.

### v1.0

- Added: `ipset` as a package dependency;
- Added: role variable `shorewall_conf`, allowing each option in the shorewall.conf file to be defined;
- Added: role variable `shorewall_package_state` to set package state of Shorewall and dependencies;
- _Changed:_ The default for `shorewall_interface` now detects the default network interface rather than fixed at `eth0` (though `eth0` is still a fall-back default);
- **Removed:** role variables: `shorewall_verbosity`, `shorewall_log_verbosity`. Use the `shorewall_conf` role variable to configure these instead.

## Author

- [Simon Bärlocher](https://sbaerlocher.ch)
- [ErmakovDmitriy](https://github.com/ErmakovDmitriy)
- [S. Veyrié](https://github.com/turb)
- [Michael Green](http://myatus.com)
- Farhad Shahbazi
- Sascha Biberhofer

## License

This project is under the MIT License. See the [LICENSE](https://sbaerlo.ch/licence) file for the full license text.

## Copyright

- Copyright (c) 2019 Simon Bärlocher
