## What is ansible-security? [![Build Status](https://secure.travis-ci.org/nickjj/ansible-security.png)](http://travis-ci.org/nickjj/ansible-security)

It is an [ansible](http://www.ansible.com/home) role to configure ssh, ufw and install fail2ban.

### What problem does it solve and why is it useful?

Getting brute forced sucks, so let's protect your servers by enabling certain security precautions through battle hardened tools.

First off we will adjust the ssh config by disabling password based logins and also disable root logins. This means that you will not be able to login without ssh keys. If you want a user role which creates a user and handles transferring ssh keys then check out my [ansible-user](https://github.com/nickjj/ansible-user) role.

Next up will be to enable and configure ufw to only expose the ports you want open. It is setup so that you can enable, disable or reset ufw by simply setting a single variable. Then you can feed it a custom list of rules to dictate which ports/protocol should be open or closed.

Lastly it installs [fail2ban](http://www.fail2ban.org/wiki/index.php/Main_Page) which is one of the best tools when it comes to rate limiting login attempts and intelligently detecting questionable activity without really knowing what you're doing (this is why I use it!).

#### Why is the ssh port still 22?

It is just too much of a hassle to change it with ansible and while it may reduce some noise on your server, it will not do anything to prevent someone from port scanning your server and finding your ssh port in a few seconds.

## Role variables

```
---
# Do you want to support ipv6?
# You may want to change this to 'no' if you have issues with ufw-init when using OpenVZ VM
security_ipv6: yes

# Values can be: enabled, disabled or reset
security_ufw_state: enabled
security_ufw_ports:
  - rule: allow             # allow or deny
    port: 80                # any port
    proto: tcp              # tcp or udp

# The amount in seconds to cache apt-update.
apt_cache_valid_time: 86400
```

## Example playbook

For the sake of this example let's assume you have a few groups and you have a typical `site.yml` file.

To use this role edit your `site.yml` file to look something like this:

```
---
- name: ensure database servers are configured
  hosts: database
  sudo: true

  roles:
    - role: nickjj.security
      tags: security
      security_ufw_ports:
        - rule: deny
          port: 80
          proto: tcp
    # add your roles here

- name: ensure app servers are configured
  hosts: app
  sudo: true

  roles:
    - { role: nickjj.security, tags: security }
    # add your roles here
```

You can set different ufw rules for each group by running the security role on each group. The reason we don't need to specify the `security_ufw_ports` variable in the **app** group is due to port 80 being open by default.

Also, let's say you want to edit the default state, you can do this by opening or creating `group_vars/all.yml` which is located relative to your `inventory` directory and then making it look something like this:

```
---
security_ufw_state: disabled
```

If you want to add multiple port rules then it should look something like this:

```
security_ufw_ports:
  - rule: allow
    port: 443
    proto: tcp
  - rule: allow
    port: 8080
    proto: tcp
  - rule: allow
    port: 4242
    proto: tcp          
```

## Installation

`$ ansible-galaxy install nickjj.security`

## Requirements

Tested on ubuntu 12.04 LTS but it should work on other versions that are similar.

## Ansible galaxy

You can find it on the official [ansible galaxy](https://galaxy.ansible.com/list#/roles/839) if you want to rate it.

## License

MIT