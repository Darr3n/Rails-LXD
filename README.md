# Rails-LXD
Ruby-On-Rails in a LXD Container

Use LXD to install development software without altering the host (well, not too much).  
The steps required to create a Demo Rails development stack upon a local installation of ubuntu 16.04:  

Software required for setup:  
Ubuntu 16.04 Desktop or server  
zfsutils-linux (will be installed by the script)  

Software to install within the container:  
RVM, which provides Ruby & Rails  
NodeJS  
Midnight Commander  

setup:  
Copy & Paste the script below into a terminal window.

# Update os:
sudo apt-get update
# Install lxd:
sudo snap install lxd
# Prepare lxd for ZFS:
sudo apt-get install zfsutils-linux 
# start the interactive configuration:
sudo lxd init
# Add user to LXD Group
sudo usermod --append --groups lxd $USER
# Within the Host, create a folder named lxd_share: 
sudo mkdir /home/$USER/lxd_share
# Allow Containers to have read, write and execute permissions to the lxd-share folder:
chmod ugo+rwx /home/$USER/lxd_share

# Create a container named rails5:
lxc launch ubuntu:16.04 rails5
# login to container:
lxc exec rails5 -- sudo --login --user ubuntu
# create a folder named lxd_share within the container Home/ubuntu folder:
mkdir /home/$USER/lxd_share
# Install Midnight Commander
sudo apt-get install mc
# Add the RVM gpg Key:
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
# Install RVM, Ruby and Rails:
\curl -sSL https://get.rvm.io | bash -s stable --ruby=2.4.2 --rails
# Add this source to Profile:
echo "source /home/$USER/.rvm/scripts/rvm" >>~/.profile
# Install NodeJS:
sudo apt-get install nodejs -y
# tidy up container:
sudo apt-get autoremove
# leave container:
exit

# Mirror folder in Container and Host:
lxc config device add rails5 homedir disk source=/home/$USER/lxd_share path=/home/ubuntu/lxd_share
# Link User Accounts:
echo "root:$UID:1" | sudo tee -a /etc/subuid /etc/subgid
# Set Read/Write access for the container:
lxc config set rails5 raw.idmap "both $UID 1000"
# Restart container:
lxc restart rails5

# Obtain the IP address of the container:
lxc list
# login to container as root user:
lxc exec rails5 -- sudo --login --user ubuntu

# goto shared folder:
cd /home/ubuntu/lxd_share
# Create Rails test application:
rails new lxdrailsdemo
# Allow the Host to have read, write and execute permissions:
chmod ugo+rwx /home/$USER/lxd_share/lxdrailsdemo
# goto the new rails folder:
cd lxdrailsdemo
# start the rails app:
rails s

That should do it, look at the output of lxc list to find the IP Address, open a browser to the rails5 container IPAddress:3000 on your host machine to see the rails info screen.


Usefull info:
mc - open midnight commander.
Create any projects within the shared lxd_share folder.
Change permissions for new folders created in the container.
Use GIT to work within the container.

