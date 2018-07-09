# Ansible Role: HAProxy

[![Build Status](https://travis-ci.org/openmicroscopy/ansible-role-haproxy.svg?branch=master)](https://travis-ci.org/openmicroscopy/ansible-role-haproxy)
[![Ansible Role](https://img.shields.io/ansible/role/14760.svg)](https://galaxy.ansible.com/openmicroscopy/haproxy/)

Installs HAProxy on RedHat/CentOS and Debian/Ubuntu Linux servers.

**Note**: This role _officially_ supports HAProxy versions 1.5 and 1.6. Future versions may require some rework.

## Requirements

If SELinux is enabled on CentOS 7 and you are using non-standard ports you must include `role: openmicroscopy.selinux-utils` before this role in your playbook.


## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    haproxy_socket: /var/lib/haproxy/stats

The socket through which HAProxy can communicate (for admin purposes or statistics). To disable/remove this directive, set `haproxy_socket: ''` (an empty string).

    haproxy_chroot: /var/lib/haproxy

The jail directory where chroot() will be performed before dropping privileges. To disable/remove this directive, set `haproxy_chroot: ''` (an empty string). Only change this if you know what you're doing!

    haproxy_user: haproxy
    haproxy_group: haproxy

The user and group under which HAProxy should run. Only change this if you know what you're doing!

    haproxy_frontends:
    - name: 'hafrontend'
      address: '*:80'
      #bind_params: 'param1 param2'
      #extra bind parameters, 'ssl' for example
      mode: 'http'
      #params:
      #  - 'some extra frontend param, acl for example'
      backend: 'habackend'
      # Optional:
      timeout client: 10s
    - name: 'hassl'
      address: 
        - '*:443'
        - '*:8443'
      bind_params: 'ssl'
      mode: 'http'
      backend: 'sslhabackend'

List of HAProxy frontends.

    haproxy_backends:
    - name: 'habackend'
      # All fields are optional apart from servers
      mode: 'http'
      balance_method: 'roundrobin'
      options:
        - "haproxy_backend_httpchk: 'HEAD / HTTP/1.1\r\nHost:localhost'"
      params:
        - 'stick on src'
      servers:
      - name: app1
        address: 192.168.0.1:80
	    extra_opts: 'inter 2s'
      - name: app2
        address: 192.168.0.2:80
      timeout connect 5s
      timeout server 20s
    - name: 'sslhabackend'
      servers:
      - name: sslapp
        address: 192.168.0.3:443

List of HAProxy backends and servers to which HAProxy will distribute requests.

    haproxy_global_vars:
      - 'ssl-default-bind-ciphers ABCD+KLMJ:...'
      - 'ssl-default-bind-options no-sslv3'

A list of extra global variables to add to the global configuration section inside `haproxy.cfg`.

Advanced users can override the template used for `haproxy.cfg` by setting `haproxy_cfg_template`. In this case most of the above role variables will be ignored unless the default template is copied.

## Dependencies

None.

## Example Playbook

    - hosts: balancer
      sudo: yes
      roles:
        - { role: openmicroscopy.haproxy }

## License

MIT / BSD

## Author Information

This role was created in 2015 by [Jeff Geerling](https://www.jeffgeerling.com/), author of [Ansible for DevOps](https://www.ansiblefordevops.com/).
