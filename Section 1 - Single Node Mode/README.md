# LXD Tutorial

Follow the instruction in official website - [Tutorial](https://documentation.ubuntu.com/lxd/en/latest/tutorial/first_steps/) and [How to access UI](https://documentation.ubuntu.com/lxd/en/latest/howto/access_ui/). 


## Preliminary

It's more reasonable to install LXD on your laptop directly. You can use it to create container and virtual machine in your daily work. However, if you want to clean up the whole environment after workshop. It's feasible to go through the whole practice in a virtual machine.The minimum hardware requirement is vcpu: 2, vram: 4G, disk: 10G

If you use Ubuntu, you can create a virtual machine by [multipass](https://multipass.run/docs)

### Multipass
```bash
snap install multipass
multipass launch 22.04 -c 2 -m 4G -d 16G -n lxd
multipass shell lxd
```

## Install and initialize LXD

1. Enter the following command to install LXD

```bash
sudo snap install  lxd --channel=5.21/stable
```
If you get an error message that the snap is already installed, run the following command to refresh it and ensure that you are running an up-to-date version:

```bash
sudo snap refresh lxd --channel=5.21/stable
```

Verify after installation
```bash
snap list
```

![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/86c17b48-e551-457c-89a0-171a3e6b3487)


2. Enter the following command to add the current user to the lxd group (the group was automatically created during the previous step):

```bash
getent group lxd | grep -qwF "$USER" || sudo usermod -aG lxd "$USER"
```

3. Enter the following command to initialize LXD:

```bash
lxd init --minimal
```

## Launch and inspect instances

You can list all images (long list) that are available on this image server with:
```bash
lxc image list ubuntu:
```

You can list the images used in this tutorial with:
```
lxc image list ubuntu: 24.04 architecture=$(uname -m)
```

1. Launch a container using the Ubuntu 22.04 image:
```
lxc launch ubuntu:22.04 ubuntu-container
```

2. Launch a VM using the Ubuntu 22.04 image:

```
lxc launch ubuntu:22.04 ubuntu-vm --vm
```

3. Check the list of instances that you launched:

```
lxc list
```

## Interact with instances

1. Run the bash command in your container:
```bash
lxc exec ubuntu-container -- bash
```

2. Enter some commands, for example, display information about the operating system:
```bash
cat /etc/*release
```

3. Exit the interactive shell:

```bash
exit
```

4. Clean up the environments

```bash
lxc stop ubuntu-container
lxc stop ubuntu-vm
lxc delete ubuntu-container
lxc delete ubuntu-vm
```

## Access the LXD web UI

1. Make sure that your LXD server is exposed to the network
```bash
lxc config set core.https_address=:8443
```

2. Enable the UI
```bash
sudo snap set lxd ui.enable=true
sudo systemctl reload snap.lxd.daemon
```

3. Access the UI in your browser by entering the address (for example, https://127.0.0.1:8443). If you build up LXD in virtual machine, you have to use the VM's ip or access the UI inside the VM.

LXD uses a self-signed certificate, which will cause a security warning in your browser. Use your browser’s mechanism to continue despite the security warning.

![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/4b9843bc-6581-4c08-a596-b827a626557e)

Bypass the security warning

![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/4ac7b433-50e0-4353-9bbe-c84e40bcf73f)

Generate the client side certificate to login
![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/7b79b8ad-5c20-4898-960c-13b7ab4b8209)

Step 1, generate the certificate and save it as lxd-ui.crt

> [!NOTE]
> You have to upload the certificate into LXD server if it's a separated VM. For example:
> ```bash
> multipass transfer lxd-ui.crt lxd:.
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


## Other reference
* https://ubuntu.com/tutorials/introduction-to-lxd-projects#1-overview
* https://ubuntu.com/tutorials/candid-authentication-lxd#1-overview
