corp.ecal network site landing page config
--

### Section 1 — preperation
1. Download the latest version of Raspbian lite from the [Raspberry Pi Foundation website](https://www.raspberrypi.org/downloads/raspbian/).
2. Write it to an SD card using [Etcher](https://www.balena.io/etcher/).
3. Create a blank file called ‘ssh’ to enable headless SSH access.

### Section 2 — updates
1. Sign in with the default credentials – `pi:raspberry`.
2. Change the default pi user password with `passwd`.
3. Update the package list with `sudo apt-get update`.
4. Process any available updates with `sudo apt-get upgrade`.
5. Finalise any updates with `sudo dpkg --configure -a`.

### Section 3 — pi configuration
1. Open raspi-config with `sudo raspi-config`.
2. Enter a new hostname in Network options > Change hostname.
3. Change the memory allocation in Advanced options > Memory split.
4. Expand the file system in Advanced options > Expand filesystem.
5. Reboot with `sudo reboot`.

### Section 4 — dynamic DNS
1. Make a new directory to keep things tidy with `mkdir cloudflare`.
2. Enter the new directory with `cd cloudflare`.
2. Copy ‘lwp-cloudflare-dyndns.sh’ to the new directory.
3. Replace ‘email@example.com’ on line 8 with the Cloudflare account email address.
4. Replace ‘global_api_key_goes_here’ on line 9 with the global API key, available under Cloudflare account settings.
5. Replace ‘example.com’ on line 10 with the root domain to be used.
6. Replace ‘home.example.com’ on line 11 with the domain or subdomain to be updated.
7. Save the file.
8. Change the file permissions to be executable with `chmod +x cloudflare/lwp-cloudflare-dyndns.sh`.
9. Run the file with `sudo sh lwp-cloudflare-dyndns.sh`. The IP will be updated and three new files should be generated.
10. Start setting up a cron job to automate the updating with `crontab -e`, then choosing a text editor.
11. Add a new line to the bottom of the crontab file (changing the timing stars as appropriate – default every five minutes) – `*/5 * * * * /bin/bash /home/pi/cloudflare/lwp-cloudflare-dyndns.sh`.
12. Restart the cron server to make sure the new job is actioned with `sudo service cron reload`.

### Section 5 — Pi-hole
1. Start the Pi-hole installer with `curl -sSL https://install.pi-hole.net | bash`.
2. Follow the Pi-hole installer.
3. Once the installer’s finished, reset the admin password for Pi-hole with `pihole -a -p`.
4. Finish setting up Pi-hole via the web interface.
5. Change the interfaces Pi-hole listens on (via Settings > DNS > Interface listening behaviour) to ‘Listen on all interfaces’.
6. Reboot with `sudo reboot`.

### Section 6 — VPN
1. Download the VPN setup script with `wget https://raw.githubusercontent.com/edwardcallow/corp.ecal-site-setup/master/vpnsetup.sh -O vpnsetup.sh`.
2. Replace ‘your pre shared key’ on line 27 with your chosen shared secret. 
3. Replace ‘your.user.name’ on line 28 with your first user’s username.
4. Replace ‘your password’ on line 29 with your first user’s password.
5. Run the installer with `sudo sh vpnsetup.sh`.
6. Open ‘/etc/iptables.rules’ and all the following lines:

> \# For IPsec/L2TP. 
> iptables -I FORWARD 2 -i ppp+ -d 192.168.0.0/24 -j ACCEPT. 
> iptables -I FORWARD 2 -s 192.168.0.0/24 -o ppp+ -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT. 
> 
> \# For IPsec/XAuth ("Cisco IPsec"). 
> iptables -I FORWARD 2 -s 192.168.43.0/24 -d 192.168.0.0/24 -j ACCEPT. 
> iptables -I FORWARD 2 -s 192.168.0.0/24 -d 192.168.43.0/24 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT. 
