Linux Run Levels:

New tools have been installed on the app server in Stratos Datacenter. Some of these tools can only be managed from the graphical user interface. Therefore, there are requirements for these app servers.

On all App servers in Stratos Datacenter change the default runlevel so that they can boot in GUI (graphical user interface) by default.

Solution
Repeat steps in other app servers

# SSH into the app server
ssh tony@stapp01

# Check current run level
sudo systemctl get-default

# Change to graphical user interface
sudo systemctl set-default graphical.target

# Check status
systemctl status graphical.target

# Start if stopped
systemctl start graphical.target

# Enable to start at boot
systemctl enable graphical.target