**Private VPN**

The first security step is adding a new user and removing root from login.

.. code-block:: bash

    dduser <name>
    usermod -aG sudo <username>
        - Deactivate root login in "sudo nano /etc/ssh/sshd_config" with "PermitRootLogin no".
        - Then "/etc/init.d/ssh restart"

* This repository has the `OpenVPN installer`_ and this `YouTube video`_ has a step by step process to the installation

.. _OpenVPN installer: https://github.com/Nyr/openvpn-install?tab=readme-ov-file

.. _YouTube video: https://www.youtube.com/watch?v=zsjBrsw9IFU&ab_channel=RackNerd

.. code-block:: bash

    scp -P 2732 papahana@papaweb.es:/LocalUser.ovpn C:\Users\ruben\Desktop\LocalUser.ovpn

.. code-block:: bash

    nano /etc/openvpn/server/server.conf

.. code-block:: bash

    sudo systemctl restart openvpn-server@server.service
