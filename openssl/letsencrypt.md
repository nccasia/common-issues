# Certbot on Centos 7 & NGINX

(certbot-auto is deprecated)

*Source*: https://certbot.eff.org/lets-encrypt/centosrhel7-nginx

1. Install `snapd`

        sudo yum install epel-release                       --- Adding EPEL to CentOS 7
        sudo yum install snapd                              --- install the snapd package
        sudo systemctl enable --now snapd.socket            --- the systemd unit that manages the main snap communication socket needs to be enabled
        sudo ln -s /var/lib/snapd/snap /snap                --- create a symbolic link between /var/lib/snapd/snap and /snap
        
2. Ensure that your version of snapd is up to date

        sudo snap install core; sudo snap refresh core
        
3. Remove certbot-auto and any Certbot OS packages
        
        sudo yum remove certbot
        
4. Install Certbot

        sudo snap install --classic certbot
        
5. Prepare the Certbot command

        sudo ln -s /snap/bin/certbot /usr/bin/certbot
        
6. Choose how you'd like to run Certbot

        sudo certbot --nginx
        sudo certbot certonly --nginx
        
7. Test automatic renewal

        sudo certbot renew --dry-run
        systemctl list-timers
        
