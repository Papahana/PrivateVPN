Private VPN
===========

Use this commands to setup the server.

.. code-block:: bash

    sudo apt-get -y remove nano
    
    sudo apt-get -y update
    sudo apt-get -y upgrade
    
    sudo apt-get -y install nano
    sudo apt install -y locate
    
    sudo updatedb

The first security step is adding a new user and removing root from login.

.. code-block:: bash

    adduser <name>
    usermod -aG sudo <username>
        - Deactivate root login in "sudo nano /etc/ssh/sshd_config" with "PermitRootLogin no".
        - Then "/etc/init.d/ssh restart"

Change default port of SSH and limit the access to only ipv4.

.. code-block:: bash

    Port <X>
    AddressFamily inet  # Access only ipv4
    PermitRootLogin no
    PasswordAuthentication no  # Only if we can use a SSH key
    PermitEmptyPasswords no

Usually the VPS provider has a built in firewall, but we can add another to be controlled inside the server.

.. code-block:: bash

     sudo apt install nftables

This command install net-tools for checking status on ports.

.. code-block:: bash

     sudo apt-get -y install net-tools

This commands are necessary to finish setting up nftables.

.. code-block:: bash

     sudo systemctl enable nftables
     sudo systemctl start nftables
     sudo systemctl status nftables

This code goes in "/etc/nftables.conf".

.. code-block:: bash

    #!/sbin/nft -f
    
    flush ruleset
    
    table inet filter {
            chain input {
                    type filter hook input priority 0; policy drop;
                    iif "lo" accept
                    ct state established,related accept
                    udp dport 3240 accept
                    iif "tun0" accept
                    tcp dport <port> accept  # SSH Port
            }
            chain forward {
                    type filter hook forward priority 0; policy drop;
                    iif "tun0" accept
                    oif "tun0" accept
            }
            chain output {
                    type filter hook output priority 0; policy accept;
            }
    }
    
    table ip nat {
            chain postrouting {
                    type nat hook postrouting priority 100;
                    iif "tun0" oif "eth0" ip saddr 0.0.0.0/0 masquerade  # IP de la interfaz VPN
            }
            chain prerouting {
                    type nat hook prerouting priority 0;
            }
    }

Every time a change is made in the file **nftables.conf**, you need to execute this command.

.. code-block:: bash

     systemctl restart nftables

This repository has the `OpenVPN installer`_ and this are default settings for the first getting started

.. _OpenVPN installer: https://github.com/Nyr/openvpn-install?tab=readme-ov-file

* Protocol that OpenVPN should use is UDP, enter the number 1 in the prompt
* The port that OpenVPN should listen to is up to you
* The DNS is recommended to be number 1
* The name is also up to you

Now press enter and the installation process starts.

To create the client file that goes in your PC we need to execute the installer once again.

* In the prompt we enter number 1, because we want to add a new client
* The name is up to you, for example "**LocalUser.ovpn**"

We need to send this file back to our computer, to do so we can use this command.

.. code-block:: bash

    scp -P <port> <user>@<domain>:/LocalUser.ovpn C:\Users\<user>\Desktop\LocalUser.ovpn

To prevent data logging from your VPN we need to change the setting **verb** from 3 to 0

.. code-block:: bash

    nano /etc/openvpn/server/server.conf
    verb 0

So this setting take effect, we can execute this command or reboot the server.

.. code-block:: bash

    sudo systemctl restart openvpn-server@server.service

**SSH Key**

On Windows PowerSheel as admin run this command to install the utility in case necessary.

.. code-block:: bash

     Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

Use this command to generate your pair of keys, if you use RSA 4096 they will be more secure. There is **no need** to enter data in the following prompt and you can leave the **password empty**.

.. code-block:: bash

     ssh-keygen -t rsa -b 4096

The Public Key goes on the server. Run this command as non root user to create the folder for the Public Key.

.. code-block:: bash

     mkdir ~/.ssh && chmod 700 ~/.ssh

Then, run this command to copy the Public Key to the server through the terminal.

.. code-block:: bash

     scp -P <SSH_PORT> $env:USERPROFILE/.ssh/id_rsa.pub <username>@<domain>:~/.ssh/authorized_keys
