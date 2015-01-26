Role Name
=========

Installs consul-template as either an upstart or systemd service.

Requirements
------------

Any pre-requisites that may not be covered by Ansible itself or the role should be mentioned here. For instance, if the role uses the EC2 module, it may be a good idea to mention in this section that the boto package is required.

Role Variables
--------------

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

Depends on ansible-consul role if consul-template needs to talk to a local Consul server.

Config
------------
If you need to add your own .cfg files for your templates (such as an nginx site conf), just put them in {{ consul_template_home }}/config/ and consul-template will merge everything in that directory into one config file.

Example: 
```
# This tells consul-template where to find and put the nginx template
template: src=nginx.cfg.j2 dest={{ consul_template_home }}/config/nginx.cfg

# This is the nginx consul-template template. Note that we are not using an Ansible 'template' for this as we don't want it interpreting the double curly brackets in the file because that is what consul-template uses
copy: src=files/nginx-sites.conf dest={{ consul_template_home }}/templates/nginx-sites.conf
```
templates/nginx.cfg.j2
```
template {
  source = "{{ consul_template_home }}/config/nginx.cfg"
  destination = "/etc/nginx/conf.d/upstreams.conf"
  command = "service nginx reload"
}
```
files/nginx-sites.conf
```
upstream app {
  least_conn;
  {{range service "production.app"}}server {{.Address}}:{{.Port}} max_fails=3 fail_timeout=60 weight=1;
  {{else}}server 127.0.0.1:65535; # force a 502{{end}}
}
```

Example Playbook
----------------

```
- hosts: consul-template-servers
  sudo: yes
  roles:
    - ansible-consul
    - ansible-consul-template
  vars:
    consul_is_ui: false
    consul_is_server: true
    consul_datacenter: "gce-us-central1-a"
    consul_bootstrap: true
    consul_bind_address: "{{ ansible_default_ipv4['address'] }}"
    use_systemd: true
    use_upstart: false
```

License
-------

Apache v2.0

Author Information
------------------

Grig Gheorghiu
http://agiletesting.blogspot.com
