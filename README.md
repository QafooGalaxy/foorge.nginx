Qafoo Nginx
===========

Ansible Role to setup Nginx as a FastCGI frontend.

It currently comes with support for php-fpm based fastcgi. This role is
required when using the "foorge.runner" with type: "fastcgi".

It does not create vhosts itselfs, this is done by "foorge.runner".

Requirements
------------

- Ubuntu Server
- Dependency on "qafoo.base" role

Role Variables
--------------

    ---
    http_config: {}
    nginx_user: "www-data"
    nginx_rotation_interval: weekly
    nginx_rotation_files: 52
    nginx_consul_ports: [80]

Example Playook
---------------

Minimal example:

    ---
    - hosts: app
      roles:
        - name: "qafoo.nginx"

Access and Error logs are always placed at `/var/log/{{ foorge_app }}/nginx/{{ item.process }}_(access|error).log` and
a logrotate script is registered to rotate weekly and keep 52 files.

LICENSE
-------

MIT License
