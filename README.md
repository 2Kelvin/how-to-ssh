# How To SSH

1. **Create SSH key pair in your personal computer**. This will create your personal SSH private and public key.
    ```bash
    ssh-keygen
    ```

2. **Copy your SSH public key to the remote server.**
From your personal PC run this command.
    ```bash
    ssh-copy-id server_user@server_ip_address
    ```

Then you'll be prompted for the **remote server's user account password** in order to complete copying your personal SSH public key to the remote server.

Example: ssh-copy-id rockylinux_user@192:168:98:22

3. **Login to server using SSH**
    ```bash
    ssh server_user@server_ip_address
    ```

Example: ssh rockylinux_user@192:168:98:22

4. ## For servers

- ### RHEL/Rocky Linux
**Use `nmcli` to manually set a static ip for the servers**
nmcli is (Network Manager CLI (command line interface))

Example:
  ```bash
  # ens160 is the ethernet of my machine: get yours by running 'nmcli device': select the device connected to the internet: ethernet or wifi
  sudo nmcli connection modify ens160 ipv4.addresses 192.168.100.200/24 ipv4.gateway 192.168.100.1

  # changing from automatic DHCP ip address to a manually set static ip address
  sudo nmcli connection modify ens160 ipv4.method manual

  # adding google dns servers: 8.8.8.8 is the default and 8.8.4.4 is the backup DNS servers
  sudo nmcli connection modify ens160 ipv4.dns 8.8.8.8,8.8.4.4

  # applying above changes to reflect new configurations
  sudo nmcli connection up ens160
  ```

- ### Ubuntu Server

**Update `netplan` config (the YAML file located in /etc/netplan/) to set a static IP address for the ubuntu server**

Example of disabling automatic IP address by DHCP and setting a static IP. I include also the router's default gateway and Google's DNS servers just like I did in rocky linux above.

    ```bash
    network:
      version: 2
      ethernets:
        ens33:
          dhcp4: no # disabling DHCP
          addresses:
            - 192.168.100.201/24 # setting a static IP address
          routes:
            - to: default
              via:  192.168.100.1 # setting the default gateway IP
          nameservers:
            addresses: [8.8.8.8, 8.8.4.4] # setting up Google's DNS servers
    ```
To apply this changes in a safe way (if any yaml typos or errors, it reverts to default configs) use this command:
  ```bash
  sudo netplan try
  ```

5. **Configure `hosts file` in your personal PC to map server IPs to custom server names**

The host file to edit is: **/etc/hosts**.
    ```bash
    sudo vim /etc/hosts
    ```

Add your custom server name to IP mapping (just like DNS does) at the bottom of the file and save
    ```bash
    # my vmware vm servers
    # mapping their IPs to custom names I can remember easily just like DNS
    192.168.100.200    rockyservervm.lab
    192.168.100.201    ubuntuservervm.lab
    ```

You can now ssh into your servers using the names you assigned them like so:
    ```bash
    ssh ubuntu_user@ubuntuservervm
    ```
    or
    ```bash
    ssh rocky_user@ubuntuservervm
    ```

6. ## Firewall (firewalld (RHEL/Rocky) and ufw (Ubuntu))
To access a web served from nginx by the server into your personal PC you have to allow this network communication past the server's firewall. You have to let `http` and `https` services through the firewall. This opens `port 80` and `port 443` respectively for web communication.

  ### RHEL/Rocky

    ```bash
    # firewall-cmd is the tool controlling firewalld service
    # my server is set on 'public' security zone ... 
    # ... un this command to check yours:  sudo firewall-cmd --get-active-zones
    # --add-service: adds the actual service you want to pass thro the firewall e.g. --add-service=http ...
    # you can add more than one service in one command by wrapping them in curly brackets and separating them with a comma with no spaces as shown below; where am adding http and https services
    # --permanent: I want this changes to be persistent thro reboots and system updates
    sudo firewall-cmd --zone=public --add-service={http,https} --permanent

    # reload firewalld service to see the new applied configurations
    sudo firewall-cmd --reload

    # check the services section to see that both http and https have been added to pass thro firewall
    sudo firewall-cmd --list-all
    ```

  ## Ubuntu

    ```bash
    # allowing ssh & http past ubuntu firewall
    # add them one by one in the terminal starting with ssh (very important)
    sudo ufw allow ssh
    sudo ufw allow http
    sudo ufw allow https

    # enabling the above set rules
    sudo ufw enable
    ```
  
    Allowing logging of ip addresses that tried to break/ unsuccessfully enter into the server. This log file is located at `/var/log/ufw.log`.

    ```bash
    sudo ufw logging on
    ```

7. **Nginx web server on browser**

If you have Nginx setup to serve a website from the server you configured above, you can access that website in a browser by using its name like so:
    ```bash
    http://rockyserver.lab
    ```