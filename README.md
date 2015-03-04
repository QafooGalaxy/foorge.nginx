Qafoo Nginx
===========

Ansible Role to setup Nginx as either Loadbalancer or FastCGI frontend.

Requirements
------------

- Ubuntu Server
- Dependency on "qafoo.base" role

Role Variables
--------------

    ---
    hosts: []
    http_config: {}
    nginx_selfsigned_subject: "/C=DE/ST=Northrhine-Westfalia/L=Bonn/O=IT/CN={{ ansible_fqdn }}"
    nginx_user: "www-data"
    nginx_default_listen: "*:80"
    nginx_rotation_interval: weekly
    nginx_rotation_files: 52
    nginx_consul_ports: [80]

Example Playook
---------------

Minimal example:

    ---
    - hosts: app
      roles:
        - name: "qafoo.base"
        - name: "qafoo.nginx"
          hosts:
            - { name: "qafoocom", server_name: "qafoo.com qafoo.local", root: "/var/www/qafoo.com" }

This sets up Nginx with one virtual host named "qafoocom", listening on
``*:80`` for http requests against the hosts `qafoo.com` or `qafoo.local`. The
default backend is PHP-FPM and defaults to listening for an `index.php` in the
root folder `/var/www/qafoo.com` using the FPM socket `/var/run/php5-fpm.sock`
that the "qafoo.php" role defaults to.

Access and Error logs are always placed at `/var/www/nginx/{{ name }}_(access|error).log` and
a logrotate script is registered to rotate weekly and keep 52 files.

    ---
    - hosts: app
      roles:
        - name: "qafoo.base"
        - name: "qafoo.nginx"
          nginx_http_config:
            proxy:
              - proxy_set_header X-Real-IP  $remote_addr
              - proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
            upstream:
              - upstream foo { server 127.0.0.1:8080 weight=10; }
          nginx_hosts:
            - name: "qafoocom"
              server_name: "qafoo.com qafoo.local"
              root: "/var/www/qafoo.com"

            - name: "symfonyapp"
              listen: "443"
              server_name: "qafoo.com"
              index: "app.php"
              root: "/var/www/symfonyapp/web"
              backend_type: "phpfpm"
              fastcgi_pass: "/var/run/php5-fpm-foo.sock"

            - name: "emptyexample"
              listen: "*:80"
              server_name: "freestyle.com"
              backend_type: "empty"
              access_log_format: "mycustom"
              error_log_format: "mycustomerr"
              directives:
                  - ssl_certificate /etc/nginx/ssl/server.crt
                  - ssl_certificate_key /etc/nginx/ssl/server.key
                  - location "/assets/toolbar" {
                        alias "/var/www/toolbar-assets"
                    }

The following variables exist on a host item:

- name: Required, the name of the server used for the filename in /etc/nginx/sites-enabled and logfiles for example.
- server_name: Required, the `server_name` directive of nginx.
- listen: Defaults to `*:80`, the IP + port this server is listening to.
- backend_type: Defaults to `phpfpm` using a fastcgi pass on php files, there is also `empty` which is only controlled by the `directives` list.
- fastcgi_pass: defaults to `/var/run/php5-fpm.sock` matching the "qafoo.php" role defaults. Only used in `backend_type: phpfpm` template.
- root: Required when backend_type is `phpfpm`, which is the default.
- access_log_format: Defaults to `combined`
- error_log_format: Defaults to `warn`
- directives: A list of strings that are added to the server.

The `http_config` is a hashmap containing a list of entries. Each hashmap key leads to a file `/etc/nginx/conf.d/key.conf`.

LICENSE
-------

MIT License
