# MicroCloud (Cluster)

Official document: https://canonical-microcloud.readthedocs-hosted.com/en/latest/how-to/install/

```bash
sudo snap install lxd --channel=5.21/stable --cohort="+"
sudo snap install microceph --channel=quincy/stable --cohort="+"
sudo snap install microovn --channel=22.03/stable --cohort="+"
sudo snap install microcloud --channel=latest/stable --cohort="+"
sudo snap list
```

Verify
```
sudo snap list
```
![image](https://github.com/chihuilu/ubuntu-workshop/assets/1013484/2a57062b-ad43-4d1b-bb16-f9683897caf4)

If lxd are installed before, please [refresh](https://canonical-microcloud.readthedocs-hosted.com/en/latest/how-to/snaps/#keep-cluster-members-in-sync) the snap.

```bash
sudo snap refresh lxd --channel=5.21/stable --cohort="+"
```

sudo microcloud init
