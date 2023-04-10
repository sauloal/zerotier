# ZeroTier

<https://hub.docker.com/r/zerotier/zerotier>

<https://hub.docker.com/r/zerotier/zerotier/tags?page=1&name=arm64>

<https://discuss.zerotier.com/t/guide-zerotier-gui-on-linux-mint-ubuntu/9447>

## Install

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

## Open port

`9993 UDP`
