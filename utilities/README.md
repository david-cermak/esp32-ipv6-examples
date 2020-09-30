# Utilities

## Running examples in containers

```
            QEMU                net utils
        |--------------|      |--------------|
        |       | tap0 |      |       radvd  |
        | esp32 | br0  |      |       dhcp6  |
        |       | eth0 |      | eth0 |       |
        |--------------|      |--------------|
                    |            |
                    +-- bridge --+
```

### Qemu container

1. Start the container with
```
docker run -it -v local_dir:/container_mapped --rm  --cap-add=NET_ADMIN qemu /bin/bash
```

2. Networking inside the container
```
ip link add br0 type bridge
mkdir /dev/net
mknod /dev/net/tun c 10 200
chmod 666 /dev/net/tun 
ip tuntap add dev tap0 mode tap
ip link set tap0 master br0
ip link set eth0 master br0
ip link set tap0 up
ip link set br0 up
```

3. Start qemu inside the container
```
qemu-system-xtensa -nographic     -machine esp32     -drive file=image.bin,if=mtd,format=raw -nic tap,model=open_eth,ifname=tap0,downscript=no,script=no
```

### net-utils container

Packages: `radvd`, `isc-dhcp-server`, see the Dockerfile

1. RA config
```
interface eth0
{
    AdvSendAdvert on;
    AdvManagedFlag off;
    AdvOtherConfigFlag on;

        prefix 2001:db8:1:1::/64
        {
                AdvOnLink on;
                AdvAutonomous on;
        };

        RDNSS 2001:db8:1:1::1 {
                AdvRDNSSLifetime 300;
                AdvRDNSSOpen off;
        };
};
```

### Bridging the containers

1. Configure the docker daemon.json:
```
{
  "debug" : true,
  "experimental" : true,
  "ipv6": true,
  "fixed-cidr-v6": "2001:db8:1::/64"
}
```

2. Start containers with `--ipv6` and network caps:
```
docker run --sysctl net.ipv6.conf.all.disable_ipv6=0 --cap-add=NET_ADMIN ....
```

3. Create network with `--ipv6` and the subnet:
```
docker network create my-net --ipv6 --subnet "2001:db8:1::/64"
```

4. Connect containers to the bridge
```
docker network connect my-net qemu_container
docker network connect my-net net_utils_container
```
