Role Name
=========

Ansible Puppet Executor is a stand alone role designed to easily bootstrap and do a local puppet execution of arbitary puppet modules, backed by arbitary hiera data (determined by ansible inventory variables). The role is broken down into three main stages

1. Bootstrapping /etc/puppet/modules on the remote nodes with puppet modules from the source of choice. The role has support for
* git
* puppetfile/r10k
* local

2. Populating the default hiera common.yaml file with contents of the variable hieradata from the ansible inventory (essentially allowing you to use the ansible inventory and all it's features to directly populate the contents of hiera and thus, puppet)

3. Running puppet with a site.pp that simply calls hiera with the value "classes", therefore running those puppet classes, with their variables populated from hiera

Requirements
------------

This role requires that puppet itself be installed on the remote nodes (through however source is preferrable). To be able to overwrite particular hiera values for different hosts via the ansible inventory, you need to set "hash_behavior=merge" in your ansible.cfg under the [defaults] section

Role Variables
--------------

This role has a number of variables to utilise how the puppet modules are sourced, what data goes into hiera, and what classes are applied via puppet. They are

__puppet_executor_modules_install_method__ This needs to be one of git, r10k, or local (defaults to local)

__puppet_executor_modules_source__ The value of this is dependant on the value of __puppet_executor_modules_install_method__, but it refers to the actual location to source the puppet modules (see examples)

__puppet_executor_hieradata__ This is a dictionary which contains key-value pairs hiera settings to be directly populated into hiera on the target hosts. If you use "hash_behavior=merge" in ansible.cfg, you can actually leverage the full power of the ansible inventory to overwrite specific values for this as needed at all levels (global, group, and individual host)

__puppet_executor_classes__ This is a list of the classes to be applied to the node. This also gets populated into a hiera variable called "classes" which the manifest references directly.

Dependencies
------------

None

Example Playbook
----------------

The following is an example playbook to source your puppet modules from a puppetfile via http (installed via r10k), then apply the ntp class to the node with servers set

    - hosts: all
      vars:
        puppet_executor_modules_install_method: puppetfile
        puppet_executor_modules_source: http://example.com/puppet/modules/Puppetfile
        puppet_executor_hieradata:
          ntp::servers: ['clock.ntp.org']
        puppet_executor_classes:
          - ntp
      roles:
        - { role: ansible-puppet-executor }

The following is an example playbook to source your puppet modules from a local directory on the server running ansible (/etc/puppet/modules), then apply the classes motd and custom1

    - hosts: all
      vars:
        puppet_executor_modules_install_method: local
        puppet_executor_modules_source: /etc/puppet/modules
        puppet_executor_hieradata:
          motd::content: "hello world"
          custom1::variable1: 1234
          custom1::variable2: "asdf1234"
        puppet_executor_classes:
          - motd
          - custom1
      roles:
        - { role: ansible-puppet-executor }

The following is an example playbook to source your puppet modules from a git repo, then run the mysql class

    - hosts: all
      vars:
        puppet_executor_modules_install_method: git
        puppet_executor_modules_source: https://github.com/example/my-puppet-modules.git
        puppet_executor_hieradata:
          mysql::server::root_password: 'superSecret'
        puppet_executor_classes:
          - mysql
      roles:
        - { role: ansible-puppet-executor }

License
-------

BSD

Author Information
------------------

Graeme Gillies

Red Hat

https://github.com/ggillies
