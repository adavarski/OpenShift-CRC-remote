```
RC_IP=$(crc ip)
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
sudo systemctl start haproxy



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

