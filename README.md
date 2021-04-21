# UPT-Oskari-Ansible
[Ansible](https://docs.ansible.com) playbooks for automatic deployment of the Urban Planning Tools (UPT) for Oskari, comprising of [Oskari](https://www.oskari.org/) version 1.55.2 and the following repositories:
* [UPT-Server-Extension](https://github.com/UPTechMX/UPT-Server-Extension), an Oskari server extension for Urban Planning Tools.
* [UPT-GUI](https://github.com/UPTechMX/UPT-GUI), a graphical user interface developed in Angular for use with UPT-Server-Extension.

## Requirements
### Target server
* A fresh installed [Ubuntu Server 18.04](https://ubuntu.com/download/server) (minimum 8 GB RAM and 30 GB of free space) with a user that is part of the `sudo` group. Below is an example of a user called `username` that is created and added to the `sudo` group. 
  ```
  adduser username
  usermod -aG sudo username
  ```
* The server is fully updated and configured with a static IP address (below is an example setting the server IP address to 192.168.1.200 on the `enp0s3` network interface, check the available interface name with `ifconfig`)
  ```
  sudo apt update && sudo apt upgrade
  sudo nano /etc/netplan/50-cloud-init.yaml
  ```
  ```
  network:
     ethernets:
         enp0s3:
             dhcp4: no
             addresses: [192.168.1.200/24]
             gateway4: 192.168.1.1
             nameservers:
                 addresses: [8.8.8.8,8.8.4.4]
     version: 2
  ```
  ```
  sudo netplan apply
  sudo reboot
  ```
* Ansible installed on the target server
  ```
  sudo apt update
  sudo apt install -y software-properties-common
  sudo apt-add-repository --yes --update ppa:ansible/ansible
  sudo apt install -y ansible
  ansible --version
    ansible 2.9.6
      config file = /etc/ansible/ansible.cfg
      configured module search path = [u'/home/user/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
      ansible python module location = /usr/lib/python2.7/dist-packages/ansible
      executable location = /usr/bin/ansible
      python version = 2.7.17 (default, Nov  7 2019, 10:07:09) [GCC 7.4.0]
  ```
  
## How to run
SSH to the target server and perform the following steps:
* Clone this repo (install `git` package if necessary)
  ```
  # clone the ansible playbook repo to /opt
  cd /opt
  # become root user
  sudo su
  git clone https://github.com/UPTechMX/UPT-Oskari-Ansible.git
  cd UPT-Oskari-Ansible
  ```
* Configure the mandatory playbook variables
  * ```nano vars_postgresql.yml```
    ```
    (Optional, recommended) postgres_user_password: change the password for the default postgres role
    ```
  * ```nano vars_oskari.yml```
    ```
    oskari_domain: set this to the IP address or domain name of the server where Oskari will be accessed (default is http://192.168.1.200:8080)
    ```
  * ```nano vars_upt.yml```
    ```
    oskari_dir: location of the oskari-server directory to which dependencies and relevant repositories will be stored
    oskari_db_pw: password for oskari user in oskaridb database
    ```
* Configure `oskari-ext.properties.j2` to direct to Distance/UrbanPerformance modules
  * ```nano roles/upt_oskari/oskari-ext.properties.j2```
  ```
  # Change the following to the appropriate settings: OSKARI_PW, IP_ADDRESS_FOR_UP_BACKEND, PORT_FOR_UP_BACKEND, IP_ADDRESS_FOR_DISTANCES_BACKEND, PORT_FOR_DISTANCES_BACKEND. upws refers to the UrbanPerformance module. stws refers to the Distance module.

  ##################################
  # UPT Modules Settings
  ##################################
  up.db.URL=jdbc:postgresql://localhost:5432/oskaridb
  up.db.password=oskari
  up.db.user=OSKARI_PW
  
  upws.db.host=IP_ADDRESS_FOR_UP_BACKEND
  upws.db.port=PORT_FOR_UP_BACKEND
  stws.db.host=IP_ADDRESS_FOR_DISTANCES_BACKEND
  stws.db.port=PORT_FOR_DISTANCES_BACKEND
  ```
* Run the playbooks
    * Recommended method: run the install all playbook to install everything in order
      * This will run the playbooks above in correct order from 1 to 4
        ```
        cd /opt/ansible-onedataportal
        # become root user
        sudo su
        ansible-playbook -K -i inventory full_install.yml
          BECOME password: enter the user's sudo password
        ```
    * Or one by one
      * Must be run in the order of the filename
        ```
        cd /opt/ansible-onedataportal
        # become root user
        sudo su
        ansible-playbook -K -i inventory 0_provision_server.yml
          BECOME password: enter the user's sudo password
        ansible-playbook -K -i inventory 1_install_postgresql.yml
        ansible-playbook -K -i inventory 2_install_oskari.yml
        ansible-playbook -K -i inventory 3_install_oskari_development.yml
        ansible-playbook -K -i inventory 4_install_upt_oskari.yml
        ```
* Some manual configuration might be needed to achieve the following (this is optional):
  * Configure firewall to further secure the server
  * Reverse proxy a (sub)domain with SSL certificate
  * Change the default passwords for Oskari/GeoServer
  * Using and updating a non-english localization
* Reboot the server
  ```
  sudo reboot
  ```
* Access the Oskari at the server's IP address or domain name on a web browser:
  * Oskari Geoportal at http://the.server.ip.address:8080
  * GeoServer admin at http://the.server.ip.address:8082/geoserver/web
* Done!
