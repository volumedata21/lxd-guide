# LXD for Containers and Virtual Machines

### Goal
1. Install LXD
2. Launch an LXC Container
3. Launch a VM
4. Launch VM with KVM capabilities
5. Setup snapshots
6. Backup a container
7. Restore a Container

### Other objectives
1. Networking with Docker
2. Setting up Netplan for a bridge network (IP Addresses)
3. Adjusting storage space
4. Allowing containers to use Docker
5. iGPU passthrough
6. Home Assistant
7. Nesting
8. Setup GUI

### Notes
1. Spacing - old tutorials might use ***lxc-start*** but now it's just ***lxc start***
2. Other version of VMs is Multipass

Official Documentation:
https://documentation.ubuntu.com/lxd
https://ubuntu.com/tutorials/how-to-run-docker-inside-lxd-containers#2-create-lxd-container

### Steps for Setup
Guide: https://documentation.ubuntu.com/lxd/stable-5.21/tutorial/first_steps/

1. `sudo apt update`
2. `sudo apt install snapd`
3. `sudo snap install lxd`
4. `getent group lxd | grep "$USER"`
5. `lxd init --minimal`
6. `lxc info | grep -FA2 'instance_types'`

##### Example on using a container   
8. `lxc image list ubuntu:`
9. `lxc launch ubuntu:24.04 first`
10. `lxc shell first`
11. `lxc copy first third`
12. `lxc list`
13. `lxc start third`
14. `lxc launch ubuntu:24.04 ubuntu-vm --vm`
15. `lxc info first`
16. `lxc snapshot first clean`
17. `lxc list first`
18. `lxc info first`
19. `lxc restore first clean`
20. `lxc delete first/clean`
21. `lxc config set first snapshots.schedule @daily`

### Letting containers run Docker
Used official guide here: https://ubuntu.com/tutorials/how-to-run-docker-inside-lxd-containers#2-create-lxd-container

Needed to run these commands:
```
lxc config set [LXC NAME] security.nesting=true security.syscalls.intercept.mknod=true security.syscalls.intercept.setxattr=true
```

### Intel iGPU Passthrough
Used this guide: https://discuss.linuxcontainers.org/t/intel-gpu-hardware-acceleration-in-container/17052

Run this command on host machine:
```
lxc config device add [container] gpu gpu gid=44
```

Can check by install VAinfo (shoutout Wundertech)
```
sudo apt install vainfo
```

Then run `vainfo` and it should let you know if it's working.

### Bridge Networking with Netplan

Example taken from [https://netplan.readthedocs.io/en/stable/examples/#how-to-configure-network-bridges](https://netplan.readthedocs.io/en/stable/examples/#how-to-configure-network-bridges)

Create a file called 99_bridge.yaml in /etc/netplan. Add the following:

```text
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: no
  bridges:
    br0:
      dhcp4: yes
      interfaces:
        - enp3s0
```
Then run this command to try and see if the netplan configuration works:
```netplan try```

If it works run 
```netplan apply```

After successfully creating a new network bridge you can attach a container by running the following command:
```
lxc config device add CONTAINER-NAME eth1 nic name=eth1 nictype=bridged parent=lxdbr0
```
This was found on this thread: https://discuss.linuxcontainers.org/t/how-to-add-a-network-interface-in-lxc/437

### Running Home Assistant as a VM
Guide for running Home Assistant as a VM using LXD: https://seanblanchfield.com/2023/05/home-assistant-os-in-lxd
