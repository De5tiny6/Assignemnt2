# Assignemnt2


#!/bin/bash
sudo apt-get update -y

# Install Apache2 and Squid
sudo apt-get install -y apache2 squid

# Check if Apache2 is already installed
if hash apache2 2>/dev/null; then
    echo "Apache2 is already their"
else
    sudo apt-get install -y apache2
    echo "Apache2 has been their"
fi

# Check if Squid is already installed
if hash squid 2>/dev/null; then
    echo "Squid is already their"
else
    sudo apt-get install -y squid
    echo "Squid has been their"
fi

# Configure the network interface for 192.168.16.21/24
sudo sed -i 's/192.168.16.20/192.168.16.21/g' /etc/hosts
sudo sed -i 's/192.168.16.20/192.168.16.21/g' /etc/netplan/01-netcfg.yaml
sudo netplan apply

# Add SSH keys for the specified users
USERS=(dennis aubrey captain snibbles brownie scooter sandy perrier cindy tiger yoda)
for user in ${USERS[@]}; do
    if ! id $user &>/dev/null; then
        sudo useradd -m -s /bin/bash $user
    fi
    if [ ! -d /home/$user/.ssh ]; then
        mkdir /home/$user/.ssh
    fi
    touch /home/$user/.ssh/authorized_keys
    echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIG4rT3vTt99Ox5kndS4HmgTrKBT8SKzhK4rhGkEVGlCI student@generic-vm" >> /home/$user/.ssh/authorized_keys
    echo "rsa-key-20210915" >> /home/$user/.ssh/authorized_keys
    chown -R $user:$user /home/$user/.ssh
    chmod 700 /home/$user/.ssh
    chmod 600 /home/$user/.ssh/authorized_keys
done

# Add sudo privileges for the 'dennis' user
if id dennis | grep -qw "sudo"; then
    echo "User 'dennis' already has sudo privileges"
else
    sudo usermod -aG sudo dennis
    echo "User 'dennis' has been given sudo privileges"
fi

# Configure UFW
sudo apt-get install -y ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow 3128/tcp
sudo ufw allow from 192.168.16.0/24 to any port 80
sudo ufw allow from 192.168.16.0/24 to any port 22
sudo ufw allow from 192.168.16.0/24 to any port 3128
sudo ufw enable

echo "All necessary configurations have been applied"
