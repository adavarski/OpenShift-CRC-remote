### Setting up CodeReady Containers (CRC) on a Remote Server (ubuntu 22.04) and Remote Access to CRC (remote OpenShift 4 development environment) from Laptop
```
### Install packages
$ echo "$USER ALL=(ALL) NOPASSWD:ALL"|sudo tee -a /etc/sudoers
$ sudo apt install qemu-kvm libvirt-daemon libvirt-daemon-system network-manager dnsmasq tinyproxy

### Setup tinyproxy
$ sudo systemctl start tinyproxy
$ sudo systemctl enable tinyproxy
$ diff /etc/tinyproxy/tinyproxy.conf /etc/tinyproxy/tinyproxy.conf.ORIG
224c224
< #Allow 127.0.0.1
---
> Allow 127.0.0.1
315d313
< ConnectPort 6443
$ sudo systemctl restart tinyproxy

### Note HAProxy
sudo apt install haproxy-y
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




### Download CRC and pull-secret in OPENSIFT directory:
$ mkdir OPENSHIFT && cd OPENSHIFT/
$ wget https://mirror.openshift.com/pub/openshift-v4/clients/crc/1.25.0/crc-linux-amd64.tar.xz
$ tar -xJf ./crc-linux-amd64.tar.xz

### Place the binaries in your $PATH using .bash_profile
$ echo "export PATH=/home/davar/OPENSHIFT/crc-linux-1.25.0-amd64:/home/davar/.crc/bin/oc:$PATH" >> ~/.bash_profile
$ source ~/.bash_profile (or relogin)
### Place the binaries in your $PATH PATH using .bashrc
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
CodeReady Containers version: 1.25.0+0e5748c8
OpenShift version: 4.7.5 (embedded in executable)
$ crc console --credentials
To login as a regular user, run 'oc login -u developer -p developer https://api.crc.testing:6443'.
To login as an admin, run 'oc login -u kubeadmin -p yiT2o-XfTVU-fr8ET-ahd5f https://api.crc.testing:6443'

### Check if CodeReady Containers work
$ oc login -u kubeadmin -p yiT2o-XfTVU-fr8ET-ahd5f https://api.crc.testing:6443
$ oc get nodes
```
#### 2.2.Setup On the Laptop

```


/etc/hosts

192.168.1.99 console-openshift-console.apps-crc.testing oauth-openshift.apps-crc.testing

https://console-openshift-console.apps-crc.testing/dashboards


Screenshots: 

davar@carbon:~/Downloads$ sudo ssh davar@devops -L 6443:console-openshift-console.apps-crc.testing:6443 


davar@carbon:~/Downloads$ oc login -u kubeadmin -p VjVIZ-rDfq3-S5Ip2-4jiLU https://devops:6443
Login successful.

You have access to 67 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
```

