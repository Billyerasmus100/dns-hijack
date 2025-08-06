# dns-hijack


############################
Step 1: Install BIND9
############################


sudo apt update
sudo apt install bind9 bind9utils bind9-doc -y

###########################################################
Step 2: Disable systemd-resolved (Required on Ubuntu 22.04)
###########################################################

sudo systemctl disable systemd-resolved --now
sudo rm /etc/resolv.conf
echo "nameserver 127.0.0.1" | sudo tee /etc/resolv.conf

#####################################
Step 3: Create Catch-All DNS Zone
#####################################

sudo nano /etc/bind/named.conf.local

----------> Add below block at the bottom:

zone "." {
    type master;
    file "/etc/bind/db.captive";
};


######################################
Step 4: Create the Wildcard Zone File
#####################################

sudo nano /etc/bind/db.captive

---------> Paste the below (adjust IP as needed):

$TTL 1h
@   IN  SOA     ns captive.localhost. (
        2025080601 ; Serial (change on every edit)
        1h         ; Refresh
        15m        ; Retry
        1w         ; Expire
        1h )       ; Minimum TTL

    IN  NS      ns

ns  IN  A       192.168.1.1

*   IN  A       192.168.1.1

##################################
Step 5: Configure BIND Options
##################################

 options {
    directory "/var/cache/bind";

    recursion no;
    allow-query { any; };
    listen-on port 53 { any; };
    allow-transfer { none; };
    dnssec-validation no;
};

################################
Step 6: Check and Restart BIND9
################################

sudo named-checkconf
sudo named-checkzone . /etc/bind/db.captive


sudo systemctl restart bind9
sudo systemctl enable named

###########################
step 7: Test DNS Hijacking
############################

dig facebook.com @127.0.0.1
