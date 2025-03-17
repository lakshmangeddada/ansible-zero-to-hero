##Setting up a communication setup between ansible control node and windows managed node.

Control-Node:
Step1: Install python3.9
    yum install python3.9

Step2: Install ansible

    yum install ansible

Step3: Install required modules.

    pip3 install "pywinrm>=0.3.0"

    pip3 install "pywinrm[credssp]"

    pip3 install "pywinrm[kerberos]"
    
    pip3 install "pywinrm[ntlm]"

Managed Node: (windows)

step1: Login to windows server and create new user and passwd. Add the user to administrator group.

    ------open powershell as admin and run the below command----

   command: net localgroup Administrators <username> /add

step2: Install WinRM in the server.

    -------run the script to install WinRM-----
    download from google: https://github.com/AlbanAndrieu/ansible-windows/blob/master/files/ConfigureRemotingForAnsible.ps1

Step3: Enable WinRM & Basic Authentication

    ---run the commands as admin in powershell----

        # Enable WinRM service
        winrm quickconfig -q

        # Allow Basic Authentication
        winrm set winrm/config/service/auth '@{Basic="true"}'

        # Allow Unencrypted Communication (if required)
        winrm set winrm/config/service '@{AllowUnencrypted="true"}'

        # Restart WinRM service
        Restart-Service WinRM

Step4: Open WinRM Port (5985) in Windows Firewall by running the command below.

    command: New-NetFirewallRule -Name "WinRM-HTTP" -DisplayName "WinRM (HTTP)" -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 5985

Step5: Allow the ansible-user to Use WinRM by running the below command

    command: winrm configSDDL default

Step6: Update Ansible Inventory (inventory.ini or /inv/hosts) in your control node.

    Edit your inventory file (inventory.ini) and add:

        [windows]
        <your-public-IP>

        [windows:vars]
        ansible_user=<username>
        ansible_password=<user-passwd>
        ansible_connection=winrm
        ansible_port=5985
        ansible_winrm_transport=basic
        ansible_winrm_server_cert_validation=ignore

Step7: Test the Connection

    command: ansible -i inventory.ini windows -m win_ping


