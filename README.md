# ZeroTier

<https://hub.docker.com/r/zerotier/zerotier>

<https://hub.docker.com/r/zerotier/zerotier/tags?page=1&name=arm64>

<https://discuss.zerotier.com/t/guide-zerotier-gui-on-linux-mint-ubuntu/9447>

## Install

### Debian

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

### Manually

```bash
sudo apt install make build-essential clang lld-14 cargo pkg-config libssl-dev

mkdir -p zerotier
cd zerotier/
wget https://github.com/zerotier/ZeroTierOne/archive/refs/tags/1.10.6.tar.gz
tar axvf 1.10.6.tar.gz
rm 1.10.6.tar.gz
cd ZeroTierOne-1.10.6/
make
make selftest
./zerotier-selftest

sudo make install
```


### Docker

```bash
#export version=1.8.10-arm64
export version=1.10.6

docker pull zerotier/zerotier:$version

export network=xxxxxxxxxxxxxxxxxxxxxx

#ZEROTIER_API_SECRET
#ZEROTIER_IDENTITY_PUBLIC
#ZEROTIER_IDENTITY_SECRET

run bash

docker run --rm -i -t --entrypoint /bin/bash zerotier/zerotier:$version
```

## Create identity

### Local

```bash

zerotier-cli generat/var/lib/zerotier-one/identity.secret /var/lib/zerotier-one/identity.public YY

```

### Docker
<https://github.com/zerotier/ZeroTierOne/blob/dev/doc/zerotier-idtool.1.md>

```bash
docker run --rm -i -t \
	--entrypoint zerotier-idtool \
	--workdir /tmp/keys \
	--volume ./zerotier/keys:/tmp/keys \
	zerotier/zerotier:$version \
	generate identity.secret identity.public YY

export name=`cat zerotier/keys/identity.public | cut -c1-10`
echo $name

sudo mv zerotier/keys/identity.public zerotier/keys/identity.public
sudo mv zerotier/keys/identity.secret zerotier/keys/identity.secret
```

## Run service

### Local

<https://docs.zerotier.com/zerotier/zerotier.conf>

<https://zerotier.atlassian.net/wiki/spaces/SD/pages/6815768/Router+Configuration+Tips>

<https://github.com/zerotier/ZeroTierOne>

```bash
export network=xxxxxxxxxxxxxx

sudo mkdir -p /var/lib/zerotier-one/networks.d
sudo touch /var/lib/zerotier-one/networks.d/$network.conf

echo -e "allowManaged=1" | sudo tee    /var/lib/zerotier-one/networks.d/$network.local.conf
echo -e "allowGlobal=1"  | sudo tee -a /var/lib/zerotier-one/networks.d/$network.local.conf
echo -e "allowDefault=1" | sudo tee -a /var/lib/zerotier-one/networks.d/$network.local.conf
echo -e "allowDNS=1"     | sudo tee -a /var/lib/zerotier-one/networks.d/$network.local.conf

cat /var/lib/zerotier-one/networks.d/xxxxxxxxxxxxxx.local.conf

sudo zerotier-one -d
sudo zerotier-cli status
sudo zerotier-cli status -j
sudo zerotier-cli info
sudo zerotier-cli info -j
sudo zerotier-cli listnetworks
sudo zerotier-cli listpeers
sudo zerotier-cli peers

sudo zerotier-cli set $network allowGlobal=1
sudo zerotier-cli set $network allowDefault=1
sudo zerotier-cli set $network allowDNS=1

#For a Linux host to route via a ZeroTier network, you may (depending on distribution) need to change a setting called rp_filter:
#sudo sysctl -w net.ipv4.conf.all.rp_filter=2
```

### Docker

```bash
#export version=1.8.10-arm64
export version=1.10.6
export network=xxxxxxxxxxxxxx
export name=`cat zerotier/keys/identity.public | cut -c1-10`
#	--volume ./zerotier/db:/var/lib/zerotier-one \
#	--env ZEROTIER_IDENTITY_PUBLIC=/tmp/keys/identity.public \
#	--env ZEROTIER_IDENTITY_SECRET=/tmp/keys/identity.secret \
#	--volume ./zerotier/keys:/tmp/keys \
docker run \
	--name zerotier \
	--rm --detach \
        --network host \
	--volume ./zerotier/keys/identity.public:/var/lib/zerotier-one/identity.public \
	--volume ./zerotier/keys/identity.secret:/var/lib/zerotier-one/identity.secret \
	--cap-add=NET_ADMIN --cap-add=NET_RAW --cap-add=SYS_ADMIN \
	--env TZ=Etc/UTC \
	--device /dev/net/tun \
	zerotier/zerotier:$version \
	$network

#	--env PUID=999 --env PGID=994 \
#	--env ZEROTIER_ONE_LOCAL_PHYS=enp0s3 \
#	--env ZEROTIER_ONE_USE_IPTABLES_NFT=false \
#	--env ZEROTIER_ONE_GATEWAY_MODE=inbound \
#	--env ZEROTIER_ONE_NETWORK_IDS=«yourDefaultNetworkID(s)» \


docker logs zerotier

docker stop zerotier
```

## Probe state

You will want to probe the control socket:

```bash
docker exec -it zerotier bash

docker exec zerotier zerotier-cli info
docker exec zerotier zerotier-cli peers
docker exec zerotier zerotier-cli listnetworks
```


## Gateway

### Masquerade

<https://dev.to/yongchanghe/set-up-a-home-server-and-access-it-from-everywhere-2j2m>

```
export inet=`sudo zerotier-cli listnetworks | tail -1 | cut -d" " -f 8`
echo $inet

sudo iptables -I FORWARD -i $inet -j ACCEPT
sudo iptables -I FORWARD -o $inet -j ACCEPT
sudo iptables -I FORWARD -i $inet -o enp0s3 -j ACCEPT
sudo iptables -t nat -I POSTROUTING -o $inet  -j MASQUERADE
sudo iptables -t nat -I POSTROUTING -o enp0s3 -j MASQUERADE

sudo     /sbin/iptables-save    | sudo tee /etc/iptables/rules.v4
sudo cat /etc/iptables/rules.v4 | sudo /sbin/iptables-restore

sudo systemctl is-enabled netfilter-persistent.service
```


### UFW

<https://gist.github.com/tjelen/0c070d343c9e6d3db2fbf57e6ceafa7c>

<https://zerotier.atlassian.net/wiki/spaces/SD/pages/7110693/Overriding+Default+Route+Full+Tunnel+Mode>

<https://sensorsiot.github.io/IOTstack/Containers/ZeroTier/>

<https://github.com/zyclonite/zerotier-docker/blob/main/README-router.md>

with:

```
curl ifconfig.me
000.000.000.000
```

`enp0s3` primary physical ETH interface

~~`10.0.0.199` the public IP on the `enp0s3` interface~~

`000.000.000.000` the public IP on the `enp0s3` interface (`curl ifconfig.me`)

`192.168.194.0/24` ZT network

`/etc/ufw/before.rules`

```
# (A) Zerotier NAT
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o ens3 -s 192.168.194.0/24 -j SNAT --to-source 000.000.000.000
COMMIT

# Don't delete these required lines, otherwise there will be errors
*filter
:ufw-before-input - [0:0]
:ufw-before-output - [0:0]
:ufw-before-forward - [0:0]
:ufw-not-local - [0:0]
# End required lines

# (B) Zerotier forwarding
-A FORWARD -i zt+ -s 192.168.194.0/24 -d 0.0.0.0/0 -j ACCEPT
-A FORWARD -i enp0s3 -s 0.0.0.0/0 -d 192.168.194.0/0 -j ACCEPT
```

`sudo sysctl -p`


## Open port

`9993`

