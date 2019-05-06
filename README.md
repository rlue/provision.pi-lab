pi-lab.provision
================

Ansible playbook for provisioning a Raspberry Pi as a home server, running:

* nginx reverse proxy + automated letsencrypt certificate renewal
* multiple nginx static web servers (blog, Jekyll theme demo site, etc.)
* ddclient (automated dynamic DNS renewal)
* syncthing (file synchronization)
* Pi-hole (DNS-level adblocking)
* restic (scheduled backups)
* Plex Media Server

### TODO

* TIG stack (Telegraf, InfluxDB, Grafana)?

Preparation
-----------

```sh
# Download Raspbian Lite
$ wget https://downloads.raspberrypi.org/raspbian_lite_latest -O raspbian.zip

# Unzip install image
$ unzip raspbian.zip
$ rm raspbian.zip

# Look up microSD card device id
$ lsblk

# Copy image to microSD card
$ sudo dd bs=4M if=<path_to_img> of=/dev/<device_id> conv=fsync

# Mount the microSD card’s boot partition (via GUI?)

# Enable ssh on new installation
$ touch /media/<username>/boot/ssh

# ...and unmount
```

Then:

1. Insert the microSD card into the Raspberry Pi.
2. Connect it to power and ethernet.
3. Identify its IP address in your router’s web UI.
4. If you have ever run another device or previous installation at the same
   IP, remove the corresponding entry from your local machine’s
   `.ssh/known_hosts` file.

Usage
-----

```sh
# Add your Raspberry Pi’s IP to a new "pi-lab" group in `/etc/ansible/hosts`
$ echo -e "[pi-lab]\n<remote_ip>\n" | sudo tee -a /etc/ansible/hosts

# Manually edit `vars/template.yml` with the appropriate values
# (use `ssh-add -L` to find your SSH key)

# Encrypt your new variables file
$ ansible-vault encrypt vars/template.yml --output=vars/vault.yml

# Provision the system
$ ansible-playbook provision.yml --ask-vault-pass
```

License
-------

© 2019 Ryan Lue. This project is licensed under the terms of the MIT license.
