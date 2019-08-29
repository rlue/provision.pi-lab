provision.pi-lab
================

Ansible playbook for provisioning a Raspberry Pi as a home server, running:

* nginx reverse proxy + automated letsencrypt certificate renewal
* multiple nginx static web servers (blog, Jekyll theme demo site, etc.)
* ddclient (automated dynamic DNS renewal)
* Pi-hole (DNS-level adblocking)
* syncthing (file synchronization)
* restic (scheduled backups)
* [a toy barcode generator app](https://github.com/rlue/nelson-scandela)

### TODO

* Plex Media Server

  **Note:** [linuxserver/docker-plex](https://github.com/linuxserver/docker-plex)
  does not officially support bridge networking mode. As a workaround, launch
  the container in host networking mode first and log into the web UI to
  generate a claim token (?), then remove and recreate the container in bridge
  networking mode.

* TIG stack (Telegraf, InfluxDB, Grafana)?

* gnubiff headless email fetcher

  * install mbsync / insert passwords into `pass`
  * install gnubiff / extract config from `secrets.tar.gpg`
  * symlink homedir archive’s maildir to `~/mail`
  * ```
    $ systemctl --user daemon-reload  # ?
    $ systemctl --user gnubiff enable
    $ systemctl --user gnubiff start
    ```

Preparation
-----------

```sh
# Download Raspbian Lite
$ wget https://downloads.raspberrypi.org/raspbian_lite_latest -O raspbian.zip
$ unzip raspbian.zip
$ rm raspbian.zip

# Copy image to microSD card (use `lsblk` to find the device_id)
$ sudo dd bs=4M if=<path_to_img> of=/dev/<device_id> conv=fsync

# Mount the microSD card’s boot partition (via GUI?)

# Enable ssh on new installation
$ touch /<path>/<to>/boot/ssh

# ...and unmount
```

Then:

1. Insert the microSD card into the Raspberry Pi.
2. Connect it to power, ethernet, and optionally, a USB hard drive.
3. Identify its IP address in your router’s web UI.
4. If you have ever run another device or previous installation at the same IP,
   remove the outdated entry from your local machine’s `.ssh/known_hosts` file.

Usage
-----

```sh
# Add your Raspberry Pi’s IP to a new "pi-lab" group in `/etc/ansible/hosts`
$ echo -e "[pi-lab]\n<raspberry_pi_ip>\n" | sudo tee -a /etc/ansible/hosts

# Manually edit `group_vars/pi-lab/template.yml` with the appropriate values
# (use `ssh-add -L` to find your SSH key)

# Encrypt your new variables file
$ ansible-vault encrypt group_vars/pi-lab/template.yml --output=group_vars/pi-lab/vault

# Run the playbook
$ ansible-playbook provision.yml --ask-vault-pass
```

### Router Setup

* **ddclient:**
  If your router provides a dynamic DNS client, it may not be necessary to run
  the `ddclient` Docker service. Be sure to configure it in your router’s
  web interface (in DD-WRT, under Setup > DDNS).

* **pi-hole:**
  To make use of the `pi-hole` Docker service, be sure to set your router’s
  DNS to the LAN IP of the Raspberry Pi. If your router supports custom
  DNSMasq configuration, be sure to add the following DNSMasq options for best
  results (in DD-WRT, under Services > Services > DNSMasq):

  ```
  dhcp-option=6,<raspberry_pi_ip>
  ```

License
-------

© 2019 Ryan Lue. This project is licensed under the terms of the MIT license.
