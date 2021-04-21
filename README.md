Ansible Role: nsedit
====================

[![Build Status](https://travis-ci.com/CesarBallardini/ansible-role-nsedit.svg?branch=main)](https://travis-ci.com/CesarBallardini/ansible-role-nsedit)

Installs nsedit on PowerDNS master node to manage zones and record in it.


Requirements
------------

A node with PowerDNS installed and configured. You can see an example on https://github.com/CesarBallardini/dns-infra


For testing:

```bash
vagrant up
ansible-galaxy install -r tests/requirements.yml --roles-path=tests/roles/ 
vagrant provision --provision-with ansible-vm
```


Role Variables
--------------

Available variables are listed below, along with default values (see defaults/main.yml):

FIXME



    powerdns_api_key: 'powerdns'


The secret API key used to configure PowerDNS.


    powerdns_api_ip: '192.168.33.10'

The IP address of 

    powerdns_api_port: '8001'
 

    powerdns_api_proto: 'http'
 

    nsedit_authdb: "/usr/local/etc/nsedit/pdns.users.sqlite3"



A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

* geerlingguy.nginx
* geerlingguy.php-versions
* geerlingguy.php


Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
        - { role: nsedit, become: true }
      vars:
        #
        # PowerDNS config needed by nsedit
        #
        powerdns_api_key: 'powerdns'
        powerdns_api_ip: '192.168.33.10'
        powerdns_api_port: '8001'
        powerdns_api_proto: 'http'
        nsedit_authdb: "/usr/local/etc/nsedit/pdns.users.sqlite3"
        
        #
        # Nginx config for nsedit vhost
        #
        nginx_vhosts:
          - listen: "443 ssl http2"
            server_name: "nsedit.infra.ballardini.com.ar"
            #server_name_redirect: "www.example.com"
            root: "/var/www/nsedit.infra.ballardini.com.ar/nsedit"
            index: "index.php index.html index.htm"
            state: "present"
            error: "500 502 503 504  /50x.html"
            template: "{{ nginx_vhost_template }}"
            filename: "nsedit.infra.ballardini.com.ar.443.com.ar.conf"
            extra_parameters: |
              location / {
                  if ($request_uri ~ ^/(.*)\.html$) {
                      return 302 /$1;
                  }
              }
              location = /50x.html {
                  root   /usr/share/nginx/html;
              }
              location ~ \.php$ {
                  fastcgi_split_path_info ^(.+\.php)(/.+)$;
                  fastcgi_pass   unix:/var/run/php/php7.4-fpm.sock;
                  fastcgi_index  index.php;
                  fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
                  include        fastcgi_params;
              }
              ssl_certificate     /etc/ssl/certs/ssl-cert-snakeoil.pem;
              ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
              ssl_protocols       TLSv1.1 TLSv1.2;
              ssl_ciphers         HIGH:!aNULL:!MD5;
          - listen: "80"
            server_name: "nsedit.infra.ballardini.com.ar"
            return: "301 https://nsedit.infra.ballardini.com.ar$request_uri"
            filename: "nsedit.infra.ballardini.com.ar.80.com.ar.conf"


License
-------

MIT / BSD

Author Information
------------------

This role was created in 2021 by César Ballardini <cesar.ballardini@gmail.com>

