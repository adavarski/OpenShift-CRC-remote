### Setting up CodeReady Containers (CRC) on a Remote Server (ubuntu 22.04) and Remote Access to CRC (remote OpenShift 4 development environment) from Laptop

Pre: RH account 

#### 1. Setup OpenShift CRC on remote host (Ubuntu 22.04 LTS, Server IP: 192.168.1.99 devops)
```

### Setup sudoers

$ echo "$USER ALL=(ALL) NOPASSWD:ALL"|sudo tee -a /etc/sudoers

### Install packages

$ sudo apt install qemu-kvm libvirt-daemon libvirt-daemon-system network-manager haproxy

### Download CRC and pull-secret in OPENSIFT-CRC directory form RH:
$ mkdir OPENSHIFT-CRC && cd OPENSHIFT-CRC/
$ wget https://mirror.openshift.com/pub/openshift-v4/clients/crc/2.33.0/crc-linux-amd64.tar.xz
$ tar -xJf ./crc-linux-amd64.tar.xz

### Place the binaries in your $PATH using .bash_profile
$ echo "export PATH=/home/davar/OPENSHIFT/crc-linux-2.33.0-amd64:/home/davar/.crc/bin/oc:$PATH" >> ~/.bash_profile
$ source ~/.bash_profile (or relogin)
### Place the binaries in your $PATH using .bashrc
$ vim ~/.bashrc
export PATH="~/.crc/bin:$PATH"
eval $(crc oc-env)
$ source ~/.bashrc 

### Setup CRC:
$ crc config set memory 12288
$ crc config set consent-telemetry no
$ crc config set pull-secret-file ~/OPENSHIFT/pull-secret.txt
Successfully configured pull-secret-file to /home/davar/OPENSHIFT/pull-secret.txt
- consent-telemetry                     : no
- memory                                : 12288
- pull-secret-file                      : /home/davar/OPENSHIFT/pull-secret.txt

### Deploy CodeReady Containers virtual machine
$ crc setup
$ crc start

### Check CRC version and credentials
$ crc version
CRC version: 2.33.0+c43b17
OpenShift version: 4.14.12
Podman version: 4.4.4

$  crc console --credentials
To login as a regular user, run 'oc login -u developer -p developer https://api.crc.testing:6443'.
To login as an admin, run 'oc login -u kubeadmin -p VjVIZ-rDfq3-S5Ip2-4jiLU https://api.crc.testing:6443'

### Check if CodeReady Containers work
$ oc login -u kubeadmin -p VjVIZ-rDfq3-S5Ip2-4jiLU https://api.crc.testing:6443
$ oc get nodes

### Setup HAProxy for remote CRC access 
sudo apt install haproxy -y
sudo cp /etc/haproxy/haproxy.cfg{,.bak}

CRC_IP=$(crc ip)
sudo tee /etc/haproxy/haproxy.cfg &>/dev/null <<EOF
global
    log /dev/log local0

defaults
    balance roundrobin
    log global
    maxconn 100
    mode tcp
    timeout connect 5s
    timeout client 500s
    timeout server 500s

listen apps
    bind 0.0.0.0:80
    server crcvm $CRC_IP:80 check

listen apps_ssl
    bind 0.0.0.0:443
    server crcvm $CRC_IP:443 check

listen api
    bind 0.0.0.0:6443
    server crcvm $CRC_IP:6443 check
EOF

sudo systemctl enable haproxy --now
sudo systemctl restart haproxy

```
#### 2.Setup On the Laptop

```
Add line /etc/hosts

192.168.1.99 devops console-openshift-console.apps-crc.testing oauth-openshift.apps-crc.testing

Browser: https://console-openshift-console.apps-crc.testing/dashboards

Screenshots: 

Install oc on laptop, make ssh tunnel (OPT) and oc login

$ sudo ssh davar@devops -L 6443:console-openshift-console.apps-crc.testing:6443 

$ oc login -u kubeadmin -p VjVIZ-rDfq3-S5Ip2-4jiLU https://devops:6443
Login successful.

You have access to 67 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".

$ oc login -u kubeadmin -p VjVIZ-rDfq3-S5Ip2-4jiLU https://console-openshift-console.apps-crc.testing:6443
Login successful.

You have access to 67 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default"

```

