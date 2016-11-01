---
layout: post
author: hagzag
tags: 
  - devops
  - Ansible
permalink: "/devops/Running-Your-Own-Ansible-Driven-CA"
published: true
title: "Running your own Ansible Driven CA"
---
# Overview & Purpose

As a preparation for running a swarm cluster in production, I needed a way to manage my Root CA and distribute the certificates between my SWARM nodes, configuring their services to use them etc etc

# A root CA

There is a bunch of posts / articles out there managing your own CA, none of them offer a free, automated solution which scales.

If running in a public DNS there ss a nice free online solution which can be configured programtically (and via [ansible module](https://docs.ansible.com/ansible/letsencrypt_module.html)) called [https://letsencrypt.org/](https://letsencrypt.org/) there are also provides which give a free official SSL certificate which expire every 3 monthes which could be also a suitable solution … 

In my case I needed a CA I can create | destroy | redistribute etc so I had in a way to create my own kind of solution ...

# CA Objectives

1. Install OpenSSL on your CA server host

    1. Configure the CA server options

    2. Generate CA private key

    3. Generate CA certificate generated with that key

2. Generate the required certificate requests for each of your nodes { including the CA server itself }

3. Distribute both the CA cert and the Host certificates to clients 

4. Configure my services to use these certs & keys

# Materials Needed

1. An inventory of hosts you wish to generate certificates for ...

2. [Ansible CA role](https://github.com/shelleg/ansible-role-ca/)

# How does this work ?

__In "shelleg context" the hosts / inventory could be either generated on the fly via a [Dynamic Inventor*y](http://docs.ansible.com/ansible/intro_dynamic_inventory.html) or via general group_vars/all/xx_hosts file (more on this in another post  …)__

* Ansible managed hosts:

Let’s take a look at a part of our group vars which hold our inventory, this example has 1 CA server and 2 nodes like so:

<script src="https://gist.github.com/hagzag/5727cd33f710bfbca2c3c6e5d556c8ea.js"></script>

* Ansible CA role -> [https://github.com/shelleg/ansible-role-ca/](https://github.com/shelleg/ansible-role-ca/) whic has the following steps:

<script src="https://gist.github.com/hagzag/2d38958efd4669b61f97624d54fa0078.js"></script>


* Setting up the CA server:

<script src="https://gist.github.com/hagzag/89e32ff019abb4713d8d8a5da5152ea0.js"></script>


* Generating the node certificates:

<script src="https://gist.github.com/hagzag/26f583b5fca83605417ddae2e883898b.js"></script>


* Fetching the keys for distribution (copy from CA server to Ansible control machine):

<script src="https://gist.github.com/hagzag/a7a4a1fefd45541f3c393a20ae5abc54.js"></script>


* Distribute the Certs & keys to the various nodes:

<script src="https://gist.github.com/hagzag/7beb8acef1fcbdbe90a76f058b4c647c.js"></script>


# Gotchas

*This role is still under development ...*

Currently running the following playbook will result in all the 6 steps unless you set the available vars to prevent them as seen in the main.yml above.

The supporting vars are:

<script src="https://gist.github.com/hagzag/d730405560c3d68a11810e78bcb5f684.js"></script>


An example playbook utilizing the CA role - in the CA server:

<script src="https://gist.github.com/hagzag/cfbc99b4e63a3beb90a50056ce3e2d48.js"></script>


On the nodes which needs certificates ...

<script src="https://gist.github.com/hagzag/9f55d18246c650213e8d9d6d017e2e7d.js"></script>


**Go ahead and give a try** and tell me what you think (open an issue if needed ;)) 

# Going forward

*Issue #1:* Control the creating of the server kay only when the existing CA kay has expired, unless force create is defined … there is a mechanism in place which needs testing ...
*Issue #2:* Add support for more hosts / groups of nodes - currently supports only the shelleg.infra and shelleg.swarm.* node groups.

Hope you enjoyed this post at least as much as I enjoyed writing this role …

Comments and findings are welcome.
