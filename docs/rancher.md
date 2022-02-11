# Rancher K3S
``` bash
cp /var/lib/rancher/k3s/server/node-token
cp /etc/rancher/k3s/k3s.yaml
```
set node ip address in k3s yaml
``` bash 
export KUBECONFIG=${PWD}/k3s.yaml
```

# Install k3os steps
## login rancher
**sudo k3os install**

?> **Tips** on screen guided install

to get the right version of k3os need to align with rancher support
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.20.15%2Bk3s1 sh -
note to join (k3s agent) url for install is ip:6443 <- port needs to be this
example URL of server: https://192.168.116.163:6443
see https://www.centlinux.com/2019/05/install-lightweight-kubernetes-k3s-with-k3os.html#point4

# post install
## enable ssh
``` bash
cd /etc/ssh
sudo vi sshd_config
```

``` 
PasswordAuthentication=yes
X11Forwarding=yes
```

**sudo service sshd restart**

## ip networking
``` bash
sudo connmanctl services
sudo connmanctl config ethernet_000c294dc063_cable --ipv4 manual 192.168.2.89 255.255.255.0 192.168.2.1 --nameservers 192.168.2.90
sudo connmanctl config ethernet_000c29eeaf0e_cable --ipv6 off

sudo vim /etc/connman/main.conf
sudo vim /var/lib/rancher/k3os/hostname 
```

# RANCHER API
``` bash
LOGINRESPONSE=`curl -s 'https://127.0.0.1/v3-public/localProviders/local?action=login' -H 'content-type: application/json' --data-binary '{"username":"admin","password":"abcde123"}' --insecure`
LOGINTOKEN=`echo $LOGINRESPONSE | jq -r .token`
APIRESPONSE=`curl -s 'https://127.0.0.1/v3/token' -H 'content-type: application/json' -H "Authorization: Bearer $LOGINTOKEN" --data-binary '{"type":"token","description":"automation"}' --insecure`
APITOKEN=`echo $APIRESPONSE | jq -r .token`
curl -v -X GET 'https://127.0.0.1/v3/cluster' -H 'content-type: application/json' -H "Authorization: Bearer $APITOKEN" --insecure | jq '.data[] | [.name, .id]'
```