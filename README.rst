Private VPN
===========

The first security step is adding a new user and removing root from login.

.. code-block:: bash

    dduser <name>
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
               iif "lo" counter accept
               tcp sport 80 accept
               tcp sport 443 accept
               tcp sport 2732 accept
               udp sport 53 accept
               udp sport 3240 accept
          }
     
          chain forward {
               type filter hook forward priority 0; policy drop;
          }
     
          chain output {
               type filter hook output priority 0; policy drop;
               oif "lo" counter accept
               tcp dport 80 accept
               tcp dport 443 accept
               udp dport 53 accept
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

Use this command to generate your pair of keys, if you use RSA 4096 they will be more secure.

.. code-block:: bash

     ssh-keygen -t rsa -b 4096

You can leave the password empty. 

The **Public Key** goes on the server. Run this command to create the folder for the Publi Key.

.. code-block:: bash

     mkdir ~/.ssh && chmod 700 ~/.ssh

Then, run this command to copy the Public Key to the server through the terminal.

.. code-block:: bash

     scp -P <SSH_PORT> $env:USERPROFILE/.ssh/id_rsa.pub <username>@<domain>:~/.ssh/authorized_keys

* `Here`_ you will find more info about this topic.

.. _Here: https://help.clouding.io/hc/es/articles/4746394002972?_gl=1*1xmvv9l*_ga*OTMwMTUwMDk3LjE2OTc4MjkxNDU.*_ga_8HM2208VHR*MTY5OTM4ODIwOS40Mi4xLjE2OTkzODgyNjEuMC4wLjA.*_fplc*Q2c5STBseFhlWDJybWQyMmNaSUVtQW1OZmtVNXJXWHdOaWMwRENmcjBER3hvc2Z1dEI2S1lXYjFZZVdNeEJqVFJTJTJGVzJhZmp5SkVvNFNDbnlzd3JBd1BaM2Vtd3pMSE5nJTJGeG13aHNnQmolMkJuZm9GaEpJTUVlR2RERGgzQnl3JTNEJTNE
