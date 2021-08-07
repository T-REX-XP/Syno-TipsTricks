# Syno-TipsTricks


**SSL certificates**

**GitLab** [source](https://stackoverflow.com/questions/31791247/enable-ssl-on-gitlab-with-docker-on-synology-nas)

I just did the installation of this and some of the answers here helped me but i still ran into problems. So i thought to share my findings:

I wanted to use the lets-encrypt certs i already had generated inside Synology DSM.

1.  Create a task scheduler (user defined script)  

cp /usr/syno/etc/certificate/system/default/privkey.pem /volume1/docker/gitlab-ce/gitlab/certs/gitlab.key
cp /usr/syno/etc/certificate/system/default/fullchain.pem /volume1/docker/gitlab-ce/gitlab/certs/gitlab.crt  
adjust to your chosen name/folder when installing gitlab (in my case "gitlab-ce")
    
2.  Create a dhparam.pem file on any machine with open ssl
    openssl dhparam -out dhparam.pem 2048    
I advice not to do this on a NAS, because it will be slow (you may increase key complexity to which ever you have patients for waiting)
    
4.  Copy the dhparam.pm to your certificats folder location inside gitlab  
    /volume1/docker/gitlab-ce/gitlab/certs/  
    adjust to your chosen name/folder when installing gitlab (in my case "gitlab-ce")
    
5.  Stop gitlab in package center (stops all tree docker containers)
    
6.  On the synology_gitlab container  
    5.1 Add the two environment variables  
    GITLAB_HTTPS=true 
    SSL_SELF_SIGNED=false  
    5.2. Change gitlab port binding (container port) from 80 to 443    

This approach will automatically at a set time (your choice in the user defined script) update you generated ssl certificate if the Synology DSM (or you manually) creates a new one. This is however not an instant update, but you can trigger it manually from the task scheduler interface. Still this approach is kind of care free for personal NAS solutions.

**UniFi SDN Controller (Ubiquity)**

    cd /usr/syno/etc/certificate/system/default/
    sudo openssl pkcs12 -export -inkey privkey.pem -in fullchain.pem -out fullchain.p12 -name unifi -password pass:unifi
    
    sudo keytool -importkeystore -deststorepass aircontrolenterprise -destkeypass aircontrolenterprise -destkeystore '/volume1/@appstore/UniFi SDN Controller/data/keystore' -srckeystore fullchain.p12 -srcstoretype PKCS12 -alias unifi -srcstorepass "unifi"
    sudo rm -f fullchain.p12
    sudo synoservice --restart "pkgctl-UniFi SDN Controller"


**HASS.io**

    rm /volume1/docker/hass.io/ssl/privkey.pem
    rm /volume1/docker/hass.io/ssl/fullchain.pem 
    sudo cp /usr/syno/etc/certificate/system/default/fullchain.pem /volume1/docker/hass.io/ssl/fullchain.pem
    sudo cp /usr/syno/etc/certificate/system/default/privkey.pem /volume1/docker/hass.io/ssl/privkey.pem
    /usr/syno/sbin/synoservice --restart pkgctl-hassio

