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
    AddressFamily inet #Access only ipv4
    PermitRootLogin no
    PasswordAuthentication no
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

* This repository has the `OpenVPN installer`_ and this `YouTube video`_ has a step by step process to the installation

.. _OpenVPN installer: https://github.com/Nyr/openvpn-install?tab=readme-ov-file

.. _YouTube video: https://www.youtube.com/watch?v=zsjBrsw9IFU&ab_channel=RackNerd

.. code-block:: bash

    scp -P 2732 papahana@papaweb.es:/LocalUser.ovpn C:\Users\ruben\Desktop\LocalUser.ovpn

.. code-block:: bash

    nano /etc/openvpn/server/server.conf

.. code-block:: bash

    sudo systemctl restart openvpn-server@server.service
