---
title:  "Kubernetes Pi"
date:   2020-01-16 00:00:00 +0000
categories: raspberrypi
tags:
    - cluster
    - raspberry pi
    - kubernetes
    - automation
classes: wide
toc: true
header: 
    overlay_image: /assets/images/Kubernetes/k8s-pi.jpg 
    overlay_filter: rgba(0, 0, 0, 0.4)
published: true

---

Building a Raspberry Pi cluster and deploying Kubernetes onto it.

# Kubernetes Pi: Getting Started

The importance of incessant learning within DevOps is paramount. The continious integration and delivery of new services, alongside the rapid evolution of the industry means it can be easy to quickly fall out of touch with new tools. Building a home lab is often cited as the best way to experiment and learn, hands on, about some of the latest technologies and computing practices that are being deployed in the cloud space. It gives you the opportunity to understand the infrastructure and processes behind a lot of the applications and services that are being run in the cloud. However, home labs do often come with a hefty price tag. With the recent developments in ARM CPUs and increasing amounts of ARM friendly distributions it meant that, for me, a [Raspberry Pi](https://hfiorillo.github.io/raspberrypi/raspbery-pi/) cluster was the perfect segway into building my own home lab.

Inspiration for this blog post came from [Jeff Geerling](https://www.jeffgeerling.com/blog) & [Alex Ellis](https://blog.alexellis.io/tag/raspberry-pi/). I highly recommend you check out their blogs on all things Raspberry Pi.

## Why build a cluster?

There are a few key reasons that come to mind:

- Mainly, it allows me to explore the concept of prototyping, clustering and parallel computing using increasingly popular technologies such as Kubernetes, Docker, Ansible and Terraform.
- There is a certain level of satisfaction that comes from having a more 'hands on' approach working with bare metal machines. Whilst less maintenance, building machines in the cloud takes away a certain sense of control over your own environment.
- Provides an easy, low cost method of building a home lab (as cheap as £90 for a 5 node cluster). The idea of buying multiple computers, linking them to a network, managing the infrastructure of those servers (i.e. power, cooling, repairs etc.), and then finding the space to store them was a big turn off for me, a newbie, when considering building a home lab.
- Understanding that although this cluster is relatively small scale, there are practical uses to such a set up. The underlying infrastructure, architecture, and design that goes into creating a cluster such as this is highly applicable even in modern enterprise environments. Companies like [Bitscope](https://www.youtube.com/watch?v=78H-4KqVvrg&ab_channel=InsideHPCReport) even build out small scale supercomputer prototypes using Raspberry Pi's to effectively demonstrate the infrastructure and management necessary for such a build.
- The Raspberry Pi itself is extremely versatile, there are various different projects that don't all resolve around building a cluster.
- It looks cool. The potential of adding more and more nodes, eventually building out something like the [SuperPi](https://www.youtube.com/watch?v=KbVcRQQ9PNw&ab_channel=OracleDevelopers) seems like something that would be fun to explore.
- A means of contributing and giving back to the community :)

## **Why Kubernetes** (also known as "k8s" or "kube")**?**

It is an open source container orchestration platform that automates many of the manual processes involved in deploying, managing and scaling containerized applications. It is maintained by the [Cloud Native Computing Foundation](https://landscape.cncf.io/category=certified-kubernetes-distribution,certified-kubernetes-hosted,certified-kubernetes-installer,special&format=card-mode&grouping=category). At only **6 years old** Kubernetes has managed to establish itself year-on-year as one of the [most consistently loved platforms](https://insights.stackoverflow.com/survey/2020#technology-most-loved-dreaded-and-wanted-platforms); one of the reasons for this is Kubernetes movement towards [infrastructure-as-data](https://cloud.google.com/blog/products/containers-kubernetes/understanding-configuration-as-data-in-kubernetes) (IaD) and away from more traditional [infrastructure-as-code](https://cloud.google.com/solutions/infrastructure-as-code) (IaC).  The ability of Kubernetes to express resources in a simple *YAML* file makes it easier for DevOps engineers to fully **express** workloads without the need to write code in a programming language. IaD creates more transparency in GitOps version control and makes scalability easy by simply altering the Horizontal Pod Autoscalers to cater to demand. [Ricardo Aravena](https://stackoverflow.blog/2020/05/29/why-kubernetes-getting-so-popular/)'s blog post does a better job at delving into the key parts of Kubernetes that make it the popular enterprise platform it is today. 

Independently managing each node in the cluster is hard work, and we don't like hard work. Kubernetes is our solution. We need software that makes it easier to run applications on the cluster; without needing to login to each Pi separately and manually running the processes ourselves. Kubernetes is installed across all of the nodes in the cluster, creating a *software defined cluster*. Where a pool of nodes known as '*agents'* (our Raspberry Pi's) are managed by a '*master*' node and application workloads are orchestrated between all these nodes. 

Put simply: The idea behind a Kubernetes cluster is that although a singular node may be able to carry out a task relatively quickly, what happens if that node fails? What happens if the load becomes too large for a singular node to handle and you want to horizontally scale? Using Kubernetes allows you to scale with ease from the command line whilst it also handles the management of a cluster; assigning out a particular task for all the nodes to work on in parallel. Allowing for the task to be carried out far more effectively than any single node could, and if one of them fails? The exisiting nodes will carry its burden.

**Distributions** 

As Kubernetes is an open source project, it makes its source code [publicly and freely available on GitHub](https://github.com/kubernetes/kubernetes). This means anyone can build their own distribution of Kubernetes - and people have. Most people turn to a Kubernetes distribution to meet their container orchestration needs, as they provide a software package with a pre-built version of Kubernetes and offer tools to help with setup and installation processes. There are a large range of distributions available. Choosing the right version of Kubernetes depends highly on your use case. Here are a few honourable mentions:

- [k8s](https://kubernetes.io/) - Standard Kubernetes (barely runs on Pi's & CPU will take a huge hit)
- [OpenShift](https://www.openshift.com/) (RedHat - Enterprise Version) - Requires 3 Master Servers with 16GB RAM each
- [Docker's Kind](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [microk8s](https://microk8s.io/) (lightweight version)
- [k0s](https://github.com/k0sproject/k0s) (NEW extremely lightweight version) - May do another cluster set up using Ubuntu & k0s
- [Rancher's k3s](https://github.com/k3s-io/k3s) (extremely lightweight version) - **Our choice**. Easier to set up for more advanced High Availability. Comes in at around 40mb and bundles all the low-level components.

In the following how-to we will focus on k3s, closely following the works of both Jeff and Alex in their various blogs on the matter. In doing so, we are going to use Ansible to help with the automation of our [install](https://github.com/k3s-io/k3s-ansible). 

[Ansible](https://www.ansible.com/overview/how-ansible-works) is an agentless automation tool that by default manages machines over the SSH protocol. This makes our job a hell of alot easier and way less tedious.

As I am personally familar with Ansible I find this method easier. But, there are a number of alternative ways in which you can install Kubernetes to the nodes:

- You can SSH into the maser node, install k3s on there. Then extract the **join token** from the master node and repeat the process for all the worker nodes whilst joining them all separately using the join token.
- Alex Ellis' [k3sup](https://github.com/alexellis/k3sup) → Bootstrapping Kubernetes with k3s on Pi (boasts full set up speed of ~1 minute)

# How-to deploy Kubernetes to your Pi:

This guide is for newbies (like me) and should be relatively straight forward. If you have any issues, persist and don't give up! Feel free to drop me a message on any of the platforms in my bio and I'll give my best attempt at troubleshooting.

## Shopping list

- 2 or more Raspberry Pi's (4 in my case)
- Multi power USB port (more Watts the better)
- USB Type-C Cables (for Raspberry Pi's)
- 32gb MicroSD cards ([make sure you get good ones!](https://www.pidramble.com/wiki/benchmarks/microsd-cards))
- Cluster Case (one with fans)

Optional:

- Network Switch with as many ports as you need, remember to have one extra slot for the ethernet-in
- 5 ethernet cables

 Other options include:

- [Raspberry Pi Cluster Hat](https://thepihut.com/products/cluster-hat-v2-0?variant=31595299045438&currency=GBP&utm_medium=product_sync&utm_source=google&utm_content=sag_organic&utm_campaign=sag_organic&gclid=Cj0KCQiA0fr_BRDaARIsAABw4Esha3l5mA2XhqAd2emTduUdZuiPYwhP4Mgk-t0sE2CArF-KOJwnO08aAieuEALw_wcB)
- [Raspberry Pi Turing Pi](https://turingpi.com/)

Total cost for my set up came to around £200, I used Raspberry Pi 4B - 1 x 4GB & 3 x 2GB.

## Assembling the cluster

Follow through the Raspberry Pi assembly process for your cluster case. If you have a network switch, connect up the Ethernet cables to your Raspberry Pi's. Connecting via ethenet allows any data-intensive applciations to exchange instruction without being hampered by wireless LAN or other network traffic, not essential but it is nice. and they should look a little something like this. 

![Raspberry Pi 4B Cluster](/assets/images/Kubernetes/picluster.jpg)

## Setting up the Operating System

My OS of choice was the official [Rasperry Pi OS Lite](https://www.raspberrypi.org/software/operating-systems/). A basic version of Raspberry Pi OS with no desktop GUI, ample for running Kubernetes as we won't be looking at the nodes' display output.

**Flashing the OS:**

This process can be done via [Etcher](https://www.balena.io/etcher/). Simply plug in your microSD card reader, flash the `.zip` image file of your chosen OS to the microSD card.

![Etcher Flashing Disk](/assets/images/Kubernetes/etcher.jpg)

**Manual process of flashing your microSD card using CLI:**

Once you have inserted your microSD card, verify the disk the card is using on your device, then unmount the disk to prepare for the following command:

```bash
diskutil list # lists the volumes 
diskutil unmountDisk /dev/disk2 # unmount the microSD 
```

Begin the flashing process:

```bash
~/Desktop/Computers/Images/2020-raspios-buster-armhf-lite.img | sudo dd bs=1ms \
of=/dev/rdisk2
```

The dd utility will directly write the contents of a disk image your microSD card, using a block size of 1mb. When the utility is finished, you will see a boot volume appear on your desktop (or in finder).

**Enabling SSH:**

Then we need to create an empty ssh file in the boot volume. Simply creating the ssh file will enable ssh on the device, as this is disabled by default: 

```bash
touch /Volumes/boot/ssh # create a file called ssh in the microSD boot vol
```

> Note: if you don't have a network switch, now is the time to add your WiFi credentials to the boot volume:

```bash
cd /Volumes/boot # change directory into the microSD boot volume
vim wpa_supplicant.conf # create a file called wpa_supplicant.conf and open in vim
```

Inside vim, paste the following and include your own Network Credentials, make sure to press 'i' to enter insert mode (the top 2 lines are only included in the latest version of Raspberry Pi OS):

```bash
country=GB # Your 2-digit country code
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="YOUR_NETWORK_NAME"
    psk="YOUR_PASSWORD"
    key_mgmt=WPA-PSK
}
```

To exit out of vim press: `Esc` then `:wq` and `Enter`.

Unmount the card one last time and your ready to go.. 

```bash
cd # change back to home dir
diskutil unmountDisk /dev/disk2 # unmount the disk
```

Or not, this process needs to be repeated for each Raspberry Pi in the cluster.

### Finding Raspberry Pi's on your network

Once all the Pi's have their subsequent microSD cards flashed, plug them in and power them on, allow them a few minutes to fully boot. Now we need to find them on our network; you can have a look on your router to see any new registered devices and their subsequent IP addresses or install a network scanner - I personally use [Angry IP Scanner](https://angryip.org/) as seen below, but you are free to use `nmap` or other networking mapping tools. Scan your network in the range of /24. For instance, my router IP was 192.168.1.1 and I scanned 192.168.1.1/24 to find all the devices on my network. The default Raspberry Pi hostname should be something like `raspberrypi.local` . Look for the corresponding IP addresses and make note.Locate all the Raspberry Pi's addresses and make note.

![Angry IP Scanner](/assets/images/Kubernetes/angryip.png)

Angry IP Scanner output

### Setting up SSH Keys

Ansible requires a [passwordless](https://www.raspberrypi.org/documentation/remote-access/ssh/passwordless.md) SSH connection in order to carry out the automated tasks on each node. To set up ssh keys for each of the nodes we can run the following commands individually:

```bash
ssh-keygen # generates ssh key on your device, follow process stated
ssh-copy-id pi@<IP-ADDRESS> # copies over your ssh key to the pi's
ssh pi@<IP-ADDRESS> # sshing into the pi
```

Change the hostname of your Raspbery Pi's to something more recognisable and relatable to their role in the cluster:

emperor - for master node

worker# - for worker nodes

> **I would also advise changing your Pi default password to avoid any unwelcome visitors. Take a look at [this](https://null-byte.wonderhowto.com/how-to/discover-attack-raspberry-pis-using-default-credentials-with-rpi-hunter-0193855/) program built especially for exploiting default credentials on Raspberry Pi's.**

```bash
sudo nano /etc/hostname # to change hostname
passwd # to change password
```

## Finally.. installing Kubernetes

There are a few prerequisites before installing Kuberentes. We need to install Ansible, Git and kubectl. If you already have them installed, skip this bit. 

Both Ansible and Git can be installed using [Homebrew](https://brew.sh/). Homebrew is a common package manager for MacOS.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install ansible git
```

To install `kubectl`, the cmd line tool that allows you to run commands against your Kubernetes cluster you can follow through the [how-to-install guide](https://kubernetes.io/docs/tasks/tools/install-kubectl/) provided on the Kubernetes website. Its quite self explanatory and easy to follow along. Oh, and if you're wondering how to pronounce kubectl, you aren't the only [one](https://medium.com/diary-of-an-sre/how-do-you-really-pronounce-kubectl-4f58f76090e5).

Next, Git clone the repository for [k3s-ansible](https://github.com/k3s-io/k3s-ansible) and walk through the usage guide.

```bash
git clone https://github.com/k3s-io/k3s-ansible
cd k3s-ansible
cp -R inventory/sample inventory/my-cluster # copies the inventory directory from sample
```

Open up the repository in a text editor and edit the Ansible inventory `hosts.ini` file to include the IP address of your 'master' and 'worker' nodes.

![Ansible hosts](/assets/images/Kubernetes/ansible-host.png)

Next, make sure to edit `inventory/my-cluster/group_vars/all.yml` to match your environment. It should look something like this: 

![Ansible all](/assets/images/Kubernetes/ansible-all.png)

Now run the following command and wait:

```bash
ansible-playbook site.yml -i inventory/my-cluster/hosts.ini
```

Once complete, grab the `kubectl` configuration from the master node and then set the `KUBECONFIG` environment variable.

```bash
scp pi@<MASTER-IP-ADDRESS>r:~/.kube/config ~/.kube/config-my-cluster # grabs the configuration file from the master node and copies into the host machine
export KUBECONFIG=~/.kube/config-my-cluster # sets KUBECONFIG equal to the config file from the master node
```

*Et Voila!* To confirm the installation was successful you can run `kubectl get nodes` . You're output should look something like this:

```bash
output:
NAME      STATUS   ROLES    AGE     VERSION
emperor   Ready    master   8m16s   v1.17.5+k3s1
worker2   Ready    <none>   7m53s   v1.17.5+k3s1
worker3   Ready    <none>   7m53s   v1.17.5+k3s1
worker1   Ready    <none>   7m50s   v1.17.5+k3s1
```

Congratulations - you're now all set up!

If you run `kubectl get pods —all-namespaces`. You will see all of the pods automatically deployed by k3s. Rancher has chosen to automatically include [Traefik](https://traefik.io/), a reverse proxy and load balancer that we can use to direct traffic into our cluster from a single entry point. In future posts, I will be exploring the use of Traefik on Kubernetes through [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) rules. For now, we can power our cluster down using Ansible  `ansible all -i inventory/hosts.ini -a "shutdown now" -b`.

Look forward to the next post, where I will be discussing the importance of monitoring your cluster and I'll walk through deploying a monitoring stack on the k3s cluster we have just created. Stay tuned!