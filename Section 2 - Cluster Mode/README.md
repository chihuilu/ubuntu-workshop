# MicroCloud (Cluster)

Follow the instruction in official website - [Tutorial](https://canonical-microcloud.readthedocs-hosted.com/en/latest/tutorial/get_started/) and [How to access UI](https://documentation.ubuntu.com/lxd/en/latest/howto/access_ui/). 


## Preliminary

The minimum number of required machines is three. vcpu: 2, vram: 4G, an additional 20G disk, an additional interface

It's more reasonable to install MicroCloud on three physical machines with in same subnet. However, if you want to clean up the whole environment after workshop. It's feasible to go through the whole practice in three virtual machine.

If you use Ubuntu, you can create three virtual machines by [LXD](https://documentation.ubuntu.com/lxd/en/latest/tutorial/first_steps/)

### Prepare 3 Virtual Machines

```bash
# Create a ZFS storage pool called disks
lxc storage create disks zfs size=100GiB

# Configure the default volume size for the disks pool
lxc storage set disks volume.size 10GiB

# Create three disks to use for remote storage
lxc storage volume create disks remote1 --type block size=20GiB
lxc storage volume create disks remote2 --type block size=20GiB
lxc storage volume create disks remote3 --type block size=20GiB

# Create a bridge network without any parameters
lxc network create microbr0

# Create the VMs, but don’t start them yet
lxc init ubuntu:22.04 micro1 --vm --config limits.cpu=2 --config limits.memory=4GiB
lxc init ubuntu:22.04 micro2 --vm --config limits.cpu=2 --config limits.memory=4GiB
lxc init ubuntu:22.04 micro3 --vm --config limits.cpu=2 --config limits.memory=4GiB

# Attach the disks to the VMs
lxc storage volume attach disks remote1 micro1
lxc storage volume attach disks remote2 micro2
lxc storage volume attach disks remote3 micro3

# Create and add network interfaces that use the dedicated MicroCloud network to each VM
lxc config device add micro1 eth1 nic network=microbr0 name=eth1
lxc config device add micro2 eth1 nic network=microbr0 name=eth1
lxc config device add micro3 eth1 nic network=microbr0 name=eth1

# Start the VMs:
lxc start micro1
lxc start micro2
lxc start micro3
```

![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/c80ec890-0b4e-46bc-a09e-70a8ca2f6654)


### Retrieve network information for MicroOVN later

Enter the following commands to find out the assigned IPv4 and IPv6 addresses for the network and note them down

```bash
lxc network get microbr0 ipv4.address
lxc network get microbr0 ipv6.address
```
![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/74c36bb6-e43b-47de-ad4d-89c1123c0567)

## Install MicroCloud on each VM

1. Configure the network interface connected to microbr0 to not accept any IP addresses (because MicroCloud requires a network interface that doesn’t have an IP address assigned):

```bash
echo 0 > /proc/sys/net/ipv6/conf/enp6s0/accept_ra
```

> [!NOTE]
> `enp6s0` is the name that the VM assigns to the network interface that we previously added as eth1.

2. Configure the network interface connected to microbr0 to not accept any IP addresses (because MicroCloud requires a network interface that doesn’t have an IP address assigned):

```bash
ip link set enp6s0 up
```

3. Install the required snaps

```bash
snap install microceph --channel=quincy/stable --cohort="+"
snap install microovn --channel=22.03/stable --cohort="+"
snap install microcloud --channel=latest/stable --cohort="+"
snap refresh lxd --channel=5.21/stable --cohort="+"
```

> [!NOTE]
> The --cohort="+" flag in the command ensures that the same version of the snap is installed on all machines.


##  Initialise MicroCloud

1. Start the initialisation process

```bash
microcloud init
```

2. Answer the questions

* Select yes to limit the search for other MicroCloud servers to the local subnet.
* Select all listed servers (these should be micro2, micro3, and micro4).

![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/d125fc0a-0215-49f6-b8aa-619abcf6900f)

* Select **no** to set up local storage.
![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/d6699e65-a2c9-40b4-8806-2d127410008c)

* Select yes to set up distributed storage.
* Select yes to confirm that there are fewer disks available than machines.
* Select all listed disks (these should be remote1, remote2, and remote3).

![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/457110df-f95e-4b91-9ac7-f1cb90e46914)

* Select yes to optionally configure the CephFS distributed file system.
* Select yes to configure distributed networking.
* Select all listed network interfaces (these should be enp6s0 on the four different VMs).
![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/2d61aa71-efb2-40e0-b571-1a93478390e0)


* Specify the IPv4 address that you noted down for your microbr0 network as the IPv4 gateway.
* Specify an IPv4 address in the address range as the first IPv4 address. For example, if your IPv4 gateway is * 192.0.2.1/24, the first address could be 192.0.2.100.
* Specify a higher IPv4 address in the range as the last IPv4 address. As we’re setting up four machines only, the range must contain a minimum of four addresses, but setting up a bigger range is more fail-safe. For example, if your IPv4 gateway is 192.0.2.1/24, the last address could be 192.0.2.254.
* Specify the IPv6 address that you noted down for your microbr0 network as the IPv6 gateway.
![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/22603b02-70a1-43de-85c3-6a73289c1805)


## Inspect your MicroCloud setup
```bash
lxc cluster list
microcloud cluster list
microceph cluster list
microovn cluster list
lxc storage list
lxc storage info remote
lxc network list
lxc network show default
```

Make sure that you can ping the virtual router within OVN. You can find the IPv4 and IPv6 addresses of the virtual router under volatile.network.ipv4.address and volatile.network.ipv6.address, respectively, in the output of lxc network show default.

![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/b4fc04e4-1ba4-4cd3-b63d-f3420f24533d)


## Launch some instances

```bash
lxc launch ubuntu:22.04 u1
lxc exec u1 -- bash
```

```bash
ping 8.8.8.8
```

## Access the UI

Access the UI in your browser by entering the address of one server (for example, https://10.158.76.8:8443).

LXD uses a self-signed certificate, which will cause a security warning in your browser. Use your browser’s mechanism to continue despite the security warning.

![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/4b9843bc-6581-4c08-a596-b827a626557e)

Bypass the security warning

![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/4ac7b433-50e0-4353-9bbe-c84e40bcf73f)

Generate the client side certificate to login
![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/7b79b8ad-5c20-4898-960c-13b7ab4b8209)

Step 1, generate the certificate and save it as lxd-ui.crt

> [!NOTE]
> You have to upload the certificate into LXD server. For example you use lxd VM as underly infra:
> ```bash
> lxc file push lxd-ui.crt micro1/root/lxd-ui.crt
> ```

Step 2, in LXD Server 

```bash
lxc config trust add lxd-ui.crt
```

In step 3, upload certificate into browser's store. Take Chrome as example:
![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/f7b4dcbe-2fb6-4e3e-ac31-bf056caff0c2)

We can select client side certificate when login the server. 
![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/c5e36866-652b-4ede-b2d0-15120e006fc1)

The first time to login
![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/f1efba5c-3e5a-4ffe-9305-830681dd442b)

Enjoy it!



