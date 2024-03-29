# ZeroTier

<https://hub.docker.com/r/zerotier/zerotier>

<https://hub.docker.com/r/zerotier/zerotier/tags?page=1&name=arm64>

<https://discuss.zerotier.com/t/guide-zerotier-gui-on-linux-mint-ubuntu/9447>

## Install

<https://askubuntu.com/questions/880304/install-zerotier-on-ubuntu-with-armhf-hardware>

```bash
#sudo sh -c 'echo "deb http://download.zerotier.com/debian/jessie jessie main #ZeroTier" > /etc/apt/sources.list.d/zerotier.list'
#wget -O - 'https://raw.githubusercontent.com/zerotier/ZeroTierOne/master/doc/contact%40zerotier.com.gpg' | sudo apt-key add -

sudo sh -c 'echo "deb [arch=amd64 signed-by=/etc/apt/trusted.gpg.d/zerotier.com.gpg] http://download.zerotier.com/debian/jessie jessie main #ZeroTier" > /etc/apt/sources.list.d/zerotier.list'
wget -O - 'https://raw.githubusercontent.com/zerotier/ZeroTierOne/master/doc/contact%40zerotier.com.gpg' | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/zerotier.com.gpg

sudo apt update
sudo apt install zerotier-one

sudo systemctl enable zerotier-one
sudo systemctl start  zerotier-one
sudo systemctl status zerotier-one

#sudo apt install net-tools
netstat -lp --numeric-ports --numeric-hosts | grep zero
```

## Create identity

```bash
zerotier-cli generat/var/lib/zerotier-one/identity.secret /var/lib/zerotier-one/identity.public YY
```

## Install IPTables persistent

```bash
sudo apt-get install iptables-persistent netfilter-persistent
```
#For a Linux host to route via a ZeroTier network, you may
(depending on distribution) need to change a setting called rp_filter:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w net.ipv4.conf.all.rp_filter=2

echo >> /etc/sysctl.conf
echo net.ipv4.ip_forward=1         >> /etc/sysctl.conf
echo net.ipv4.conf.all.rp_filter=2 >> /etc/sysctl.conf
```

## Run service

<https://docs.zerotier.com/zerotier/zerotier.conf>

<https://zerotier.atlassian.net/wiki/spaces/SD/pages/6815768/Router+Configuration+Tips>

<https://github.com/zerotier/ZeroTierOne>

```bash
export network=`zerotier-cli listnetworks | head -2 | tail -1 | gawk '{ print $3 }'`
echo "NETWORK NAME ${network}"

# for multiple networks, set manually
#export network=xxxxxxxxxxxxxx

sudo mkdir -p /var/lib/zerotier-one/networks.d

sudo zerotier-one -d
sudo zerotier-cli status
sudo zerotier-cli status -j
sudo zerotier-cli info
sudo zerotier-cli info -j
sudo zerotier-cli listnetworks
sudo zerotier-cli listpeers
sudo zerotier-cli peers

sudo zerotier-cli set $network allowManaged=1
sudo zerotier-cli set $network allowGlobal=1
sudo zerotier-cli set $network allowDefault=1
sudo zerotier-cli set $network allowDNS=1

sudo zerotier-cli get $network allowManaged
sudo zerotier-cli get $network allowGlobal
sudo zerotier-cli get $network allowDefault
sudo zerotier-cli get $network allowDNS
```

Or Manually

```bash
sudo touch /var/lib/zerotier-one/networks.d/$network.conf

echo -e "allowManaged=1" | sudo tee    /var/lib/zerotier-one/networks.d/$network.local.conf
echo -e "allowGlobal=1"  | sudo tee -a /var/lib/zerotier-one/networks.d/$network.local.conf
echo -e "allowDefault=1" | sudo tee -a /var/lib/zerotier-one/networks.d/$network.local.conf
echo -e "allowDNS=1"     | sudo tee -a /var/lib/zerotier-one/networks.d/$network.local.conf

cat /var/lib/zerotier-one/networks.d/$network.local.conf
```

## Gateway

<https://dev.to/yongchanghe/set-up-a-home-server-and-access-it-from-everywhere-2j2m>

```bash
export inet=`sudo zerotier-cli listnetworks | tail -1 | cut -d" " -f 8`
echo "ZEROTIER INET $inet"

export dftl=`ip r | grep default | gawk '{print $5}'`
echo "DEFAULT  INET $dftl"

sudo iptables -I FORWARD -i $inet   -j ACCEPT
sudo iptables -I FORWARD -o $inet   -j ACCEPT
sudo iptables -I FORWARD -i $inet   -o ${dftl} -j ACCEPT
sudo iptables -t nat -I POSTROUTING -o ${dftl} -j MASQUERADE
sudo iptables -t nat -I POSTROUTING -o eth0    -j MASQUERADE

sudo     /sbin/iptables-save    | sudo tee /etc/iptables/rules.v4
sudo cat /etc/iptables/rules.v4 | sudo /sbin/iptables-restore
sudo     /sbin/iptables-save

sudo systemctl is-enabled netfilter-persistent.service

sudo sysctl -p
```

<https://gist.github.com/tjelen/0c070d343c9e6d3db2fbf57e6ceafa7c>

<https://zerotier.atlassian.net/wiki/spaces/SD/pages/7110693/Overriding+Default+Route+Full+Tunnel+Mode>

<https://sensorsiot.github.io/IOTstack/Containers/ZeroTier/>

<https://github.com/zyclonite/zerotier-docker/blob/main/README-router.md>

## Open port

Enable port `9993` `UDP`.

```bash
ufw allow 9993/udp
```
