# ＭicroCloud Workshop

## Minimum Requirement

  * 記憶體 12+G, 可用硬碟空間 30G
  * 筆電安裝 Virtual Box 或 Hyper V 或 VM work station 虛擬平台 ，預先安裝 3 台 Ubuntu 22.04 VM
  * 每一 Ubuntu VM vcpu: 2, vram: 4G, disk: 10G

### Platform for virtual machine (Multipass)

Official document: https://multipass.run/docs/how-to-guides

Other reference for `how to use multipass`
* https://computingforgeeks.com/run-ubuntu-virtual-machines-on-linux-macos-using-multipass/
* https://www.how2shout.com/linux/how-to-install-mutliple-ubuntu-vms-using-multipass-on-ubunut-20-04/

Please use **lower** case for the VM's name. It will be used by hostname and upper case may cause some problem in k8s.

```bash
multipass launch 22.04 -c 2 -m 4G -d 16G -n microcloud-1
multipass launch 22.04 -c 2 -m 4G -d 16G -n microcloud-2
multipass launch 22.04 -c 2 -m 4G -d 16G -n microcloud-3
multipass list
multipass shell microcloud-1
```
After enter the virtual machine, please verify the network
```bash
ping 8.8.8.8
```

Troubleshooting
if public network is not available, it may be the default FORWARD policy is DROP in iptable. Setup the iptables on your laptop.

```bash
sudo iptables -L
sudo apt-get install iptables-persistent
sudo iptables -P FORWARD ACCEPT
sudo iptables -L
```
