corp.ecal-site-setup
--
corp.ecal network site server config

### Section 1: Preperation
1. Download the latest version of Raspbian lite from the [Raspberry Pi Foundation website](https://www.raspberrypi.org/downloads/raspbian/).
2. Write it to an SD card using [Etcher](https://www.balena.io/etcher/).
3. Create a blank file called ‘ssh’ to enable headless SSH access.

### Section 2: Updates
1. Sign in with the default credentials – `pi:raspberry`.
2. Change the default pi user password with `passwd`.
3. Update the package list with `sudo apt-get update`.
4. Process any available updates with `sudo apt-get upgrade`.
5. Finalise any updates with `sudo dpkg --configure -a`.

### Section 3: Pi configuration
1. Open raspi-config with `sudo raspi-config`.
2. Enter a new hostname in Network options > Change hostname.
3. Change the memory allocation in Advanced options > Memory split.
4. Expand the file system in Advanced options > Expand filesystem.
5. Reboot with `sudo reboot`.

### Section 4: Dynamic DNS
1. Make a new directory to keep things tidy with `mkdir cloudflare`.
2. Enter the new directory with `cd cloudflare`.
2. Copy ‘lwp-cloudflare-dyndns.sh’ to the new directory.
3. Replace ‘email@example.com’ on line 8 with the Cloudflare account email address.
4. Replace ‘global_api_key_goes_here’ on line 9 with the global API key, available under Cloudflare account settings.
5. Replace ‘example.com’ on line 10 with the root domain to be used.
6. Replace ‘home.example.com’ on line 11 with the domain or subdomain to be updated.
7. Save the file.
8. Change the file permissions to be executable with `chmod +x lwp-cloudflare-dyndns.sh`.
9. Run the file with `sudo sh lwp-cloudflare-dyndns.sh`. The IP will be updated and three new files should be generated.
10. Start setting up a cron job to automate the updating with `crontab -e`, then choosing a text editor.
11. Add a new line to the bottom of the crontab file (changing the timing stars as appropriate – default every five minutes) – `*/5 * * * * /bin/bash /home/pi/cloudflare/lwp-cloudflare-dyndns.sh`.
12. Restart the cron server to make sure the new job is actioned with `sudo service cron reload`.

### Section 5: Pi-hole
1. Make a new directory to keep things tidy with `mkdir vpn`.
2. Start the Pi-hole installer with `curl -sSL https://install.pi-hole.net | bash`.
3. Follow the Pi-hole installer.
4. Once the installer’s finished, reset the admin password for Pi-hole with `pihole -a -p`.
5. Finish setting up Pi-hole via the web interface.
6. Change the interfaces Pi-hole listens on (via Settings > DNS > Interface listening behaviour) to ‘Listen on all interfaces’.
7. Reboot with `sudo reboot`.

### Section 6: VPN
1. Download the VPN setup script with `wget https://raw.githubusercontent.com/edwardcallow/corp.ecal-site-setup/master/vpnsetup.sh -O vpn/vpnsetup.sh`.
2. Replace ‘your pre shared key’ on line 27 with your chosen shared secret. 
3. Replace ‘your.user.name’ on line 28 with your first user’s username.
4. Replace ‘your password’ on line 29 with your first user’s password.
5. Run the installer with `sudo sh vpnsetup.sh`.
6. Open ‘/etc/iptables.rules’ and add the following lines to the end (replacing ‘172.16.10.0/16’ with the IP range and subnet for your network):

	\# For IPsec/L2TP  
	iptables -I FORWARD 2 -i ppp+ -d 172.16.0.0/16 -j ACCEPT  
	iptables -I FORWARD 2 -s 172.16.0.0/16 -o ppp+ -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT  
	
	\# For IPsec/XAuth ("Cisco IPsec")  
	iptables -I FORWARD 2 -s 192.168.43.0/24 -d 172.16.0.0/16 -j ACCEPT  
	iptables -I FORWARD 2 -s 172.16.0.0/16 -d 192.168.43.0/24 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT  

7. Reboot with `sudo reboot`.

### Section 7: SSL
1. Install certbot with `sudo apt-get install certbot`.
2. Run the installer in webroot mode with `sudo certbot certonly --webroot`.
3. Follow the Certbot installer.
4. Generate a combined certificate and private key file by running the following command (replacing ‘pihole.example.com’ with the domain or subdomain to be used):

	sudo cat /etc/letsencrypt/live/pihole.example.com/privkey.pem \  
	/etc/letsencrypt/live/pihole.example.com/cert.pem | \  
	sudo tee /etc/letsencrypt/live/pihole.example.com/combined.pem

5. Make sure the lighttpd user (www-data) can read the certificates with `sudo chown www-data -R /etc/letsencrypt/live`.
6. Open ‘/etc/lighttpd/external.conf’ and add the contents of `https://raw.githubusercontent.com/edwardcallow/corp.ecal-site-setup/master/external.conf` to the end (replacing ‘pihole.example.com’ with the domain or subdomain to be used).
7. Restart the web server with `sudo service lighttpd restart`.
8. Reboot with `sudo reboot`.

### Section 8: Landing page
1. Download the [Landing page template](https://github.com/edwardcallow/corp.ecal-landing) with `git clone https://github.com/edwardcallow/corp.ecal-landing.git /var/www/html`.
2. Enter the web server root directory with `cd /var/www/html`.
3. Delete the unnecessary readme with `sudo rm readme.md`.
4. Open and make any relevant changes to ‘landing.php’.
5. Choose an accent colour in ‘style.css’.

### Optional steps

#### Port forwarding
1. Forward the following ports to the server via the router administration page:

	TCP port 80 (HTTP)  
	TCP port 443 (HTTPS)  
	UDP port 500 (IPSec)  
	UDP port 4500 (IPSec)  

2. Test the forwarding and web server by attempting to access the landing page and Pi-hole admin page from an external network.

#### Manage VPN users

##### Add or edit a user
1. Download the script with `wget https://raw.githubusercontent.com/edwardcallow/corp.ecal-site-setup/master/add_vpn_user.sh`.
2. Run the script with `sudo sh add_vpn_user.sh 'username.to.add' 'new.password'`.

##### Delete a user
1. Download the script with `wget https://raw.githubusercontent.com/edwardcallow/corp.ecal-site-setup/master/del_vpn_user.sh`.
2. Run the script with `sudo sh del_vpn_user.sh 'username.to.delete'`.

#### Change the shared secret
1. Open ‘/etc/ipsec.secrets’ and change the listed key in quotemarks.
2. Restart services with `service ipsec restart` and `service xl2tp restart`.

### For reference
- [Cloudflare dynamic DNS configuration](https://letswp.io/cloudflare-as-dynamic-dns-raspberry-pi/)
- [Pi and Pi-hole configuration](https://gist.github.com/bnutz/89eabf55335869444b61060e18cea4e5)
- [IPSec configuration](https://github.com/hwdsl2/setup-ipsec-vpn)
- [SSL configuration (part 1)](https://certbot.eff.org/lets-encrypt/debianbuster-other)
- [SSL configuration (part 2)](https://discourse.pi-hole.net/t/enabling-https-for-your-pi-hole-web-interface/5771)
- [Managing VPN users](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/manage-users.md)
- [Configuring VPN clients for XAuth IPSec](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients-xauth.md)
- [Configuring VPN clients for L2TP IPSec](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/docs/clients.md)