# Introduction

When you first create a new Ubuntu 20.04 server, you should perform some important configuration steps as part of the initial setup. These steps will increase the security and usability of your server, and will give you a solid foundation for subsequent actions.

## Step 1 — Logging in as root

To log into your server, you will need to know your server’s public IP address. You will also need the password or — if you installed an SSH key for authentication — the private key for the root user’s account. If you have not already logged into your server, you may want to follow our guide on how to Connect to Droplets with SSH, which covers this process in detail.

If you are not already connected to your server, log in now as the root user using the following command (substitute the highlighted portion of the command with your server’s public IP address):

    ssh root@your_server_ip

Accept the warning about host authenticity if it appears. If you are using password authentication, provide your root password to log in. If you are using an SSH key that is passphrase protected, you may be prompted to enter the passphrase the first time you use the key each session. If this is your first time logging into the server with a password, you may also be prompted to change the root password.

## Step 2 — Creating a New User

Once you are logged in as the root user, the first thing you should do is create a new user account with sudo privileges. This user account should not be the root user, and it should not have sudo privileges. Instead, you should create a new user with sudo privileges and use that account for day-to-day administrative tasks.

To create a new user account, use the adduser command and follow the prompts to set a strong password and provide the other requested information:

    adduser sammy

You will be prompted to enter a new UNIX password. Type a strong password and press Enter. You will then be prompted to enter the password again to verify it. Type the password again and press Enter.

## Step 3 — Granting Administrative Privileges

Once the new user account has been created, you can grant administrative privileges to the new user by adding it to the sudo group. To do this, use the usermod command with the -aG option to add the new user to the sudo group:

    usermod -aG sudo sammy

## Step 4 - Setting Up a Basic Firewall

A firewall is a system that controls the incoming and outgoing network traffic to your server. It can be used to block traffic to certain ports, and it can be used to limit the number of connections from a single IP address. A firewall can also be used to limit the number of failed login attempts, which can help to mitigate brute force attacks against your server.

Ubuntu 20.04 uses the UFW firewall by default. UFW provides a user-friendly way to create an IPv4 or IPv6 host-based firewall. It provides a command line interface and aims to be uncomplicated and easy to use.

To install UFW, use the apt package manager with the install command and the -y flag to automatically answer yes to any prompts:

    apt install ufw -y

Once UFW is installed, you can enable it by typing:

    ufw enable

You will be prompted to confirm that you want to enable the firewall. Type y and press Enter to continue.

## Step 5 — Allowing SSH Connections

By default, UFW blocks all incoming connections. To allow incoming SSH connections, you must tell UFW to allow them. To do this, use the ufw allow command with the OpenSSH service:

    ufw allow OpenSSH

You can verify that the rule has been added by typing:

    ufw status

You should see output similar to the following:

    Status: active

    To                         Action      From
    --                         ------      ----
    OpenSSH                    ALLOW       Anywhere
    OpenSSH (v6)               ALLOW       Anywhere (v6)

## Step 6 — Allowing HTTP and HTTPS Connections (Optional)

If you plan to host a website on your server, you will need to allow HTTP and HTTPS connections. To do this, use the ufw allow command with the HTTP and HTTPS services:

    ufw allow 'Nginx HTTP'
    ufw allow 'Nginx HTTPS'

You can verify that the rules have been added by typing:

        ufw status

You should see output similar to the following:
    
        Status: active
    
        To                         Action      From
        --                         ------      ----
        OpenSSH                    ALLOW       Anywhere
        OpenSSH (v6)               ALLOW       Anywhere (v6)
        Nginx HTTP                 ALLOW       Anywhere
        Nginx HTTP (v6)            ALLOW       Anywhere (v6)
        Nginx HTTPS                ALLOW       Anywhere
        Nginx HTTPS (v6)           ALLOW       Anywhere (v6)

## Step 7 — Enable SSH Key Authentication

By default, Ubuntu 20.04 servers are configured to use password authentication for SSH connections. This means that you can log in to your server by providing the password for the root user’s account. However, password authentication is less secure than key-based authentication, which uses a public/private key pair to authenticate users.

To enable key-based authentication, you will need to generate an SSH key pair on your local computer. If you already have an SSH key pair that you would like to use, you can skip this step.

To generate an SSH key pair, use the ssh-keygen command. When prompted to specify the file in which to save the key, press Enter to use the default location. When prompted to enter a passphrase, press Enter to use an empty passphrase:

    ssh-keygen

You will then be prompted to enter a passphrase again. Press Enter to use an empty passphrase.

Once you have generated an SSH key pair, you can copy the public key to your server. To do this, use the ssh-copy-id:

    ssh-copy-id sammy@your_server_ip

You may see the following message:

    The authenticity of host 'your_server_ip (your_server_ip)' can't be established.
    ECDSA key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
    Are you sure you want to continue connecting (yes/no/[fingerprint])?

Type yes and press Enter to continue.

You will then be prompted to enter the password for the new user’s account. Enter the password and press Enter.

Once you have copied the public key to your server, you can disable password authentication. To do this, use the nano text editor to open the /etc/ssh/sshd_config file:

    nano /etc/ssh/sshd_config

Find the PasswordAuthentication line and change the yes value to no:
    
    PasswordAuthentication no

Save and close the file when you are finished.

To apply the changes, restart the SSH service:

    systemctl restart ssh

## Step 8 — Disabling Root Login 

By default, Ubuntu 20.04 servers are configured to allow root logins over SSH. This means that you can log in to your server as the root user by providing the password for the root user’s account. However, logging in as the root user is less secure than logging in as a regular user with sudo privileges.

To disable root logins, use the nano text editor to open the /etc/ssh/sshd_config file:

    nano /etc/ssh/sshd_config

Find the PermitRootLogin line and change the yes value to no:
        
        PermitRootLogin no

Save and close the file when you are finished.

To apply the changes, restart the SSH service:

    systemctl restart ssh

## Step 9 — Enabling Automatic Updates (Optional)

Ubuntu 20.04 uses the unattended-upgrades package to automatically install security updates. This package is installed by default, but it is not enabled by default. To enable it, use the dpkg-reconfigure command with the unattended-upgrades package:

    dpkg-reconfigure -plow unattended-upgrades

You will be prompted to select the update behavior. Press the spacebar to select the No automatic updates option, and then press Enter to continue.

You will then be prompted to select the update origin. Press the spacebar to select the All configured origins option, and then press Enter to continue.

You will then be prompted to select the update package types. Press the spacebar to select the Security updates option, and then press Enter to continue.

You will then be prompted to select the update severity. Press the spacebar to select the Important and above option, and then press Enter to continue.

You will then be prompted to select the update schedule. Press the spacebar to select the Daily option, and then press Enter to continue.


## Step 10 — Installing Fail2Ban (Optional)

Fail2Ban is an intrusion prevention software framework that protects computer servers from brute-force attacks. It does this by monitoring log files and banning IP addresses that show the malicious signs — too many password failures, seeking for exploits, etc. Generally Fail2Ban is then used to update firewall rules to reject the IP addresses for a specified amount of time, although any arbitrary other action (e.g. sending an email) could also be configured. Out of the box Fail2Ban comes with filters for various services (apache, courier, ssh, etc).

To install Fail2Ban, use the apt package manager with the install command and the -y flag to automatically answer yes to any prompts:

    apt install fail2ban -y

Once Fail2Ban is installed, you can enable it by typing:

    systemctl enable fail2ban

You will be prompted to confirm that you want to enable the firewall. Type y and press Enter to continue.

## Step 11 - Finding your passphrase in MacOs Keychain, if lost (Optional)

If you are using a Mac, you can find your passphrase in the Keychain Access app. To do this, open the Keychain Access app and search for the name of your private key file. You should see an entry for the private key file. Double-click the entry to open it. You will then see the passphrase for the private key file.


