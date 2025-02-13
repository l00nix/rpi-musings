# 3 & 4 - Putting it all together - Building the cluster
After what felt like an eternity (2+ years) I had all the components together to finally build my LIM cluster.

## Hardware Build
1. The first step was to assemble the eight cluster nodes.

{% include image.html
            img="/images/limcluster_build/limcluster1.png"
            title="limcluster_build1"
            caption="Fig 6: Depiction of all the parts making up a cluster node" %}

Parts List:
1.	LiM CM4 Cluster carrier board
2.	Raspberry Pi Compute Module (CM) 4 Light 8GB RAM with WiFi - CM4108000
3.	[128x32 i2c OLED display](https://amzn.to/42TUp2B){:target="_blank"}
4.	[128GB 2232 M.2 M-key socket NVMe PCIe SSD drive](https://amzn.to/3IpXgrK){:target="_blank"}
5.	[CM4 Heat sink with thermal compound strip and screws](https://amzn.to/3BKyjDI){:target="_blank"}
6.	Custom designed 90 degree i2c OLED angle bracket
7.	[Brush less LED cooling fan CPU with screws](https://amzn.to/42VcxZJ){:target="_blank"}
8.	Not pictured – [M2.5 brass standoffs with screws](https://amzn.to/43eXom0){:target="_blank"}

Assembly line, the parts to put all eight cluster nodes together:

{% include image.html
            img="/images/limcluster_build/limcluster2.png"
            title="limcluster_build2"
            caption="Fig 7: Parts pre cluster node assembly" %}

And this is now how eight of the cluster nodes look like put together:

{% include image.html
            img="/images/limcluster_build/limcluster3.png"
            title="limcluster_build3"
            caption="Fig 8: Eight cluster nodes fully assembled" %}

Assembled cluster with eight cluster nodes in the Pi (Pringles) Tray standing on top of a 8 port PoE router:

{% include image.html
            img="/images/limcluster_build/limcluster4.png"
            title="limcluster_build4"
            caption="Fig 9: Complete cluster set up" %}

{% include image.html
            img="/images/limcluster_build/limcluster5.png"
            title="limcluster_build5"
            caption="Fig 10: Cluster Top View" %}

## Software Build
Since I wanted to use Rancher to manage the cluster, I also prepared an [Intel based UP 4000 Board](https://up-board.org/up-4000/){:target="_blank"} and installed Ubuntu 22.04 and Rancher 2.5 on it (more on the Rancher install further down). I set this up as the ‘clustercontroller’ – 10.0.0.200. All the management of the cluster (Rancher, Ansible) is done from the clustercontroller node.

So, for the most part to install the cluster software I followed NetworkChuck’s video tutorial ["i built a Raspberry Pi SUPER COMPUTER!! // ft. Kubernetes (k3s cluster w/ Rancher)"](https://www.youtube.com/watch?v=X9fSMGkjtug). I wanted to automate the cluster build as much as I could, so where possible I created Ansible scripts. But first, I needed to install [Raspberry Pi OS](https://www.raspberrypi.com/software/){:target="_blank"} on the cluster nodes.

{:start="1"}
1. Imaging
Since the cluster nodes are built with SSD drives, I used the Raspberry Pi Imager to write the OS to the eight SSD drives. 


{% include image.html
            img="/images/limcluster_build/limcluster6.png"
            title="limcluster_build6"
            caption="Fig 11: Raspberry Pi Imager"
	    url="https://www.raspberrypi.com/software/" %}

Initial set up/configuration: 

-	Enabled SSH
-	I did not enable wireless as all nodes are using network cables connected to a PoE router which also power the nodes.
-	I did not add “arm_64bit=1” to the config.txt file as I was already installing Raspberry Pi OS Bullseye 64-bit.
-	All I changed in between imaging the next node was the hostname.
-	From my test cluster set up I configured my ubiquity Dream machine to assign a static IP to each of the cluster nodes based on MAC address so I could easily identify the cluster nodes on my network. The IP address and hostname schema looks as follows:
	* Cluster node 1: 10.0.0.201 – clusternode1
	* Cluster node 2: 10.0.0.202 – clusternode2
	* …
	* Cluster node8: 10.0.0.208 – clusternode8

{:start="2"}
2. Once I booted up all eight cluster nodes I copied ssh-keys across so I can communicate and administer the nodes from the clustercontroller. I created the following Ansible hosts file in /etc/ansible/:

```bash
[clustercontroller]
10.0.0.200

[clustermaster]
10.0.0.201

[clusterworkers]
10.0.0.202
10.0.0.203
10.0.0.204
10.0.0.205
10.0.0.206
10.0.0.207
10.0.0.208

[limcluster]
10.0.0.201
10.0.0.202
10.0.0.203
10.0.0.204
10.0.0.205
10.0.0.206
10.0.0.207
10.0.0.208

[testnode]
10.0.0.209
```

Then I tested it by pinging the cluster with Ansible:

{% include image.html
            img="/images/limcluster_build/limcluster_ping.gif"
            title="limcluster_build7"
            caption="Fig 12: Pinging all cluster nodes with Ansible" %}

{:start="3"}
3. Next, I updated the OS on all cluster nodes with the following Ansible script:

```yaml
---
- hosts: "{{ "{{" }} variable_hosts {{ }} }}"
  remote_user: pi
  become: true
  become_user: root
  gather_facts: False

  tasks:
    - name: Update apt repo and cache on all Debian/Ubuntu boxes
      apt:
        update_cache: yes
        force_apt_get: yes
        cache_valid_time: 3600

    - name: Upgrade all packages on servers
      apt:
        upgrade: dist
        force_apt_get: yes

    - name: Check if a reboot is needed on all servers
      register: reboot_required_file
      stat:
        path: /var/run/reboot-required
        get_md5: no

    - name: Reboot the server if kernel updated
      reboot:
        msg: "Reboot initiated by Ansible for kernel updates"
        connect_timeout: 5
        reboot_timeout: 300
        pre_reboot_delay: 0
        post_reboot_delay: 30
        test_command: uptime
      when: reboot_required_file.stat.exists

```

{% include image.html
            img="/images/limcluster_build/limcluster_upgrade.gif"
            title="limcluster_build8"
            caption="Fig 13: Updating and rebooting Raspian OS on all cluster nodes with Ansible" %}

{:start="4"}
4. To add ```“cgroup_memory=1 cgroup_enable=memory”``` I wrote the following Ansible script so I can easily do this across multiple cluster nodes and also easily add additional nodes if wanted/needed:

```yaml
---
- hosts: "{{ "{{" }} variable_hosts {{ }} }}"
  remote_user: pi
  become: true
  become_user: root
  tasks:
    - name: Check whether /boot/cmdline.txt contains 'cgroup_memory' and append recommended cluster node vars if not found
      command: "grep  'cgroup_memory' /boot/cmdline.txt"
      register: checkmyconf
      check_mode: no
      ignore_errors: yes
      changed_when: no
      failed_when: false
    - meta: end_host
      when: checkmyconf.rc == 0

    - name: Add cgroup_memory to /boot/cmdline.txt
      ansible.builtin.lineinfile:
        path: "/boot/cmdline.txt"
        backrefs: true
        regexp: '^(.*rootwait.*)$'
        line: '\1 cgroup_memory=1 cgroup_enable=memory'
      register: updated
      when: checkmyconf.rc != 0

    - name: Reboot when /boot/cmdline.txt was updated with recommended cluster node vars
      reboot:
        connect_timeout: 5
        reboot_timeout: 300
        pre_reboot_delay: 0
        post_reboot_delay: 30
        test_command: uptime
      when: updated.failed == false
```

{% include image.html
            img="/images/limcluster_build/limcluster_append_cmdline.gif"
            title="limcluster_build9"
            caption="Fig 14: Adding additional parameters to the Raspian OS configuration with Ansible" %}

{:start="5"}
5. I DID NOT follow 'Step 2 – K3s Prep' of NetworkChuck’s tutorial and skipped the configuration of legacy IP tables. I continued NetworkChuck’s tutorial with the installation of the masternode of the k3s cluster. The k3s master node install is a 'one liner', an Ansible script is maybe a bit overkill but because I can, created the following Ansible script for the installation of the masternode:

```yaml
---
- hosts: "{{ "{{" }} variable_hosts {{ }} }}"
  remote_user: pi
  become: true
  become_user: root
  tasks:
    - name: Check if k3s is already installed
      register: k3s_installed
      stat: path=/usr/local/bin/k3s get_md5=no

    - name: k3s is already installed and exit
      debug:
        msg: "k3s is already installed on {{ "{{" }} ansible_hostname }}"
      when: k3s_installed.stat.exists

    - meta: end_host
      when: k3s_installed.stat.exists

    - name: Install k3s on master node remote target
      shell: curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.21.1+k3s1" K3S_KUBECONFIG_MODE="644" sh -s –
```

{% include image.html
            img="/images/limcluster_build/limcluster_k3s_masternode_install.gif"
            title="limcluster_build10"
            caption="Fig 15: Installing k3s on the master node with Ansible" %}

{:start="6"}
6. Next, on to installing the cluster nodes with this Ansible script:

```yaml
- hosts: "{{ "{{" }} variable_master {{ }} }}"
  gather_facts: false
  user: pi
  become: true
  become_user: root
  tasks:
     - name: "Read k3s cluster master token"
       shell: |
         cat /var/lib/rancher/k3s/server/node-token
       register: file_content
     - name: "Add k3s cluster master token to dummy host"
       add_host:
         name: "master_token_holder"
         hash: "{{ "{{" }} file_content.stdout }}"
         ip: "{{ "{{" }} inventory_hostname }}"

- hosts: "{{ "{{" }} variable_worker {{ }} }}"
  user: pi
  tasks:
    - name: Check if k3s is already installed
      register: k3s_installed
      stat: path=/usr/local/bin/k3s get_md5=no

    - name: k3s is already installed and exit
      debug:
        msg: "k3s is already installed on ""{{ "{{" }} ansible_hostname }}"
      when: k3s_installed.stat.exists

    - meta: end_host
      when: k3s_installed.stat.exists

    - name: Install k3s worker on remote target
      shell: curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION="v1.21.1+k3s1" K3S_KUBECONFIG_MODE="644" K3S_TOKEN="{{ "{{" }} hostvars['master_token_holder']['hash'] }}" K3S_URL="https://"{{ "{{" }} hostvars['master_token_holder']['ip'] }}:6443" K3S_NODE_NAME="{{ "{{" }} ansible_hostname }}" sh -

```

{% include image.html
            img="/images/limcluster_build/limcluster_k3s_workernodes_install.gif"
            title="limcluster_build11"
            caption="Fig 16: Installing k3s on the worker nodes with Ansible" %}

{:start="7"}
7. After I had the cluster installed and running, I followed NetworkChuck's instructions to install Rancher 2.5 on my clustercontoller

{% include image.html
            img="/images/limcluster_build/rancher.png"
            title="limcluster_build12"
            caption="Fig 17: Rancher log in screen after install on the clustercontroller" %}

{:start="8"}
8. Once Rancher was installed, I connected the limcluster to it

{% include image.html
            img="/images/limcluster_build/rancher_complete.gif"
            title="limcluster_build13"
            caption="Fig 18: Importing the k3s cluster into Rancher" %}

{% include image.html
            img="/images/limcluster_build/rancher_screenshot.png"
            title="limcluster_build14"
            caption="Fig 19: Cluster statistics displayed in Rancher Dashboard" %}

{% include image.html
            img="/images/limcluster_build/rancher_screenshot1.png"
            title="limcluster_build15"
            caption="Fig 20: Cluster statistics displayed in Rancher 'Cluster Explorer'" %}

I now had a functioning 8 node k3s cluster set up, managed by Rancher.
