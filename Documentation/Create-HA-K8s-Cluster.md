![logo](https://eliasdh.com/assets/media/images/logo-github.png)
# 💙🤍Create HA K8s Cluster🤍💙

## 📘Table of Contents

1. [📘Table of Contents](#📘table-of-contents)
2. [🖖Introduction](#🖖introduction)
3. [✨Steps](#✨steps)
    1. [👉Step 1: Set up basic configuration](#👉step-1-set-up-basic-configuration)
    2. [👉Step 2: Install and Configure Docker](#👉step-2-install-and-configure-docker)
    3. [👉Step 3: Install and Configure Kubernetes](#👉step-3-install-and-configure-kubernetes)
    4. [👉Step 4: Set up reverse/forward proxy](#👉step-4-set-up-reverseforward-proxy)
    5. [👉Step 5: Initialize the Kubernetes cluster](#👉step-5-initialize-the-kubernetes-cluster)
4. [📝Notes](#📝notes)
5. [🔗Links](#🔗links)

---

## 🖖Introduction

This document describes the steps to create a Kubernetes cluster with nodes. The OS is Ubuntu 24.04 LTS.
- [Youtube Video](https://www.youtube.com/watch?v=vX2n05t0AQg)
- [Github Repository - A must read](https://github.com/kubernetes/kubeadm/blob/main/docs/ha-considerations.md#options-for-software-load-balancing)

## ✨Steps

### 👉Step 1: Set up the operating system

- Install the necessary software packages. **For all nodes & proxy**
```bash
sudo apt-get update -y && sudo apt-get upgrade -y
```

- Set time zone to `Europe/Brussels` on both servers. **For all nodes & proxy**
```bash
sudo timedatectl set-timezone Europe/Brussels
```

- Set the hostname of the servers. **For all nodes & proxy**
```bash
sudo hostnamectl set-hostname node00 # e.g. node01, node02, node03, ...
```

- Add the following lines to `/etc/motd` to display a welcome message when logging in. **For all nodes & proxy**
```bash
sudo nano /etc/motd
```
```text
??????????????????????????????????????????JJJJJJJJJJJJJJJJ??????????????????????????????????????????
??????????????????????????????????JJJJJ???77!!!~~~~~~!!77???JJJJ????????????????????????????????????
??????????????????????????????JJJ??7~^::..               ..:^^~7??JJJ???????????????????????????????
???????????????????????????JJJ7!^:                              .:^!7JJJ????????????????????????????
????????????????????????JJ?7~:                                       :~7?J??????????????????????????
??????????????????????JJ?~:                                             .~?JJ???????????????????????
????????????????????J?!:.                                                 .~?JJ?????????????????????
??????????????????JJ7:                                                       ^7J????????????????????
?????????????????J?^                     ..:^~~~~~~~~^::.                      ~?J??????????????????
????????????????J!.                  .:~7???JJJJJJJJJJJ??7!~:.                  .7J?????????????????
??????????????J?^                 :^!?JJJ????????????????JJJJ?!^.                 ~J????????????????
?????????????J?:                ^7?JJ?????????????????????????JJ?~.                ~J???????????????
????????????J?:               :7JJ????????JJJ????????JJJJ????????J?!.               ~J??????????????
????????????J^              .~?J????????J?7~:.......:^~!7??????????J?^               ~J?????????????
???????????J!               7J????????J?~.              .^7??????????J~               7J????????????
??????????J?.              !J????????J7.             .^!???????????JJJ?.              :?????????????
??????????J~              ^J????????J!            :~7?JJ???????JJJ?7~:                 ?J???????????
???????????.              7J???????J7         .^~7?JJ???????JJ??!^.                .:~7?????????????
?????????J7              .????????J?.      .^!?JJJ??????JJJ?!~:                 :~!?JJJ?????????????
???????????.             .????????J^    :~7?JJ???????JJ?7~:.                .^!7?JJ?????????????????
???????????.             .?????????..:!??JJ??????JJJ?!^.                .^!7?JJJ????????????????????
??????????J^              7J?????????JJJ?????JJJ?7~:.               .:~7?JJJ???????JJ???????????????
??????????J!              7J??????????????JJ?7!^.               .:~7?JJJ???????JJJ?7~::?????????????
????????????.           :!J???????????????7^:                :~!?JJJ????????JJ??!^.   :?????????????
???????????J^       .^!??J?????????????J!.               :~!??JJ????????JJJ?7~:       7J????????????
???????????J7   .:~7?JJJ??????JJJ????????7~:.        .^!??JJJ???????JJJ?7~:.         :J?????????????
?????????????!!!?JJJ???????JJJ?77?JJJ????JJJ?77777777?JJJ???????JJJ?7~:.             7J?????????????
?????????????JJJ????????JJ?7~:.  .:!?JJJ?????JJJJJJJJ??????JJJ??7~:.                ~J??????????????
?????????????????????JJ?7~:          :~7?JJJJ????????JJJJJ?7!^:.                   ^J???????????????
???????????????????J?7~.                .:~7?????????77~^:.                       ~?????????????????
??????????????????J!.                        .::::::.                           :7J?????????????????
????????????????????~.                                                        ^!?J??????????????????
????????????????????J?~.                                                   :~7JJ????????????????????
??????????????????????J?7^                                              .~7?JJ??????????????????????
????????????????????????JJ?~:                                        :~7?JJ?????????????????????????
??????????????????????????JJ?7~^.                                .^!7?JJ????????????????????????????
?????????????????????????????JJJ??7!~^::..                 ..:^!7?JJJ???????????????????????????????
?????????????????????????????????JJJJJJ???77!~~~~~~~!!!!!77??JJJJ???????????????????????????????????
??????????????????????????????????????????JJJJJJJJJJJJJJJJJ?????????????????????????????????????????
????????????????????????????????????????????????????????????????????????????????????????????????????
------------------------------
- Welcome to server [node00] -
------------------------------
```

- Disable cloud-init on the servers. **For all nodes & proxy**
```bash
echo "network: {config: disabled}" | sudo tee /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
sudo touch /etc/cloud/cloud-init.disabled
```

- Configure the network interfaces of the servers. **For all nodes & proxy**
```bash
sudo rm -rf /etc/netplan/50-cloud-init.yaml
sudo nano /etc/netplan/01-netcfg.yaml
```
```yaml
network:
  version: 2
  ethernets:
    ens18:
      dhcp4: no
      dhcp6: no
      accept-ra: no
      addresses:
        - 192.168.1.170/24 # e.g. 192.168.1.171, 192.168.1.172, 192.168.1.173, ...
      nameservers:
        addresses:
          - 192.168.1.120
          - 8.8.8.8
      routes:
        - to: default
          via: 192.168.1.1
```

- Disable IPv6 on the servers. **For all nodes & proxy**
```bash
echo -e "net.ipv6.conf.all.disable_ipv6 = 1\nnet.ipv6.conf.default.disable_ipv6 = 1\nnet.ipv6.conf.lo.disable_ipv6 = 1" | sudo tee /etc/sysctl.conf | sudo sysctl -p
```

- Add the hostnames to the `/etc/hosts` file. **For all nodes & proxy**
```bash
echo -e "127.0.0.1 localhost\n192.168.1.170 proxy1\n192.168.1.171 node01\n192.168.1.172 node02\n192.168.1.173 node03\n192.168.1.174 node04\n192.168.1.175 node05\n192.168.1.176 node06\n192.168.1.177 node07\n192.168.1.178 node08\n192.168.1.179 node09" | sudo tee /etc/hosts > /dev/null
```

- Apply the network configuration. **For all nodes & proxy**
```bash
sudo chmod 600 /etc/netplan/01-netcfg.yaml
sudo chown root:root /etc/netplan/01-netcfg.yaml
sudo netplan apply
```

- Reboot the servers. **For all nodes & proxy**
```bash
sudo reboot
```

### 👉Step 2: Install and Configure Docker

- Install dependencies. **For all nodes**
```bash
sudo apt-get install -y apt-transport-https gnupg ca-certificates curl software-properties-common
```	

- Check for missing packages. **For all nodes**
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```	

- Add Docker's official GPG key. **For all nodes**
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

- Add the repository to Apt sources. **For all nodes**
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

- Install Docker. **For all nodes**
```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

- Add user to the Docker group. **For all nodes**
```bash
sudo usermod -aG docker $USER
exit
```

- Verify the installation. **For all nodes**
```bash
docker --version
```

### 👉Step 3: Install and Configure Kubernetes

- Disable swap on the node. **For all nodes**
```bash
sudo swapoff -a
sudo sed -i 's|/swap.img|#/swap.img|g' /etc/fstab
```

- Load the necessary kernel modules. **For all nodes**
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/k8s.conf
```

- Enable IP forwarding on the node. **For all nodes**
```bash
echo -e "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/k8s.conf
sudo sysctl --system
```

- Configure the container runtime. **For all nodes**
```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo sed -i 's|SystemdCgroup = false|SystemdCgroup = true|g' /etc/containerd/config.toml
sudo systemctl restart containerd
```

- Install Kubernetes. **For all nodes**
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```

- Install Kubernetes tools. **For all nodes**
```bash
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

- Reboot the servers. **For all nodes**
```bash
sudo shutdown -h now
```

> **Note:** Assuming you did this in a virtual machine, you can convert it to a template and then, create a few instances. e.g. node01, node02, node03, ...

| ID  | Name   | Roll        | IP            | CPU | RAM   | Disk | OS               | Type |
|-----|--------|-------------|---------------| --- | ----- | ---- | -----------------| ---- |
| 170 | proxy1 | Proxy       | 192.168.1.170 | 1   | 0.5GB | 8GB  | Ubuntu 24.04 LTS | CT   |
| 171 | node01 | Master      | 192.168.1.171 | 2   | 2GB   | 32GB | Ubuntu 24.04 LTS | VM   |
| 172 | node02 | Master      | 192.168.1.172 | 2   | 2GB   | 32GB | Ubuntu 24.04 LTS | VM   |
| 173 | node03 | Master      | 192.168.1.173 | 2   | 2GB   | 32GB | Ubuntu 24.04 LTS | VM   |
| 174 | node04 | Worker      | 192.168.1.174 | 2   | 4GB   | 32GB | Ubuntu 24.04 LTS | VM   |
| 175 | node05 | Worker      | 192.168.1.175 | 2   | 4GB   | 32GB | Ubuntu 24.04 LTS | VM   |
| 176 | node06 | Worker      | 192.168.1.176 | 2   | 4GB   | 32GB | Ubuntu 24.04 LTS | VM   |
| 177 | node07 | Worker      | 192.168.1.177 | 2   | 4GB   | 32GB | Ubuntu 24.04 LTS | VM   |
| 178 | node08 | Worker      | 192.168.1.178 | 2   | 4GB   | 32GB | Ubuntu 24.04 LTS | VM   |
| 179 | node09 | Worker      | 192.168.1.179 | 2   | 4GB   | 32GB | Ubuntu 24.04 LTS | VM   |

### 👉Step 4: Set up reverse/forward proxy

- Create a new user on the proxy server. **proxy1**
```bash
sudo useradd -m -s /bin/bash -G sudo elias
sudo passwd elias
```

- Install [keepalived](https://www.keepalived.org/) and [haproxy](https://www.haproxy.org/) on the proxy server. **proxy1**
```bash
sudo apt-get install -y keepalived haproxy
```

- Configure the keepalived service. **proxy1**
```bash
sudo rm -rf /etc/keepalived/keepalived.conf
sudo nano /etc/keepalived/keepalived.conf
```
```conf
global_defs {
    router_id proxy1
    script_user root
    script_security 1
}

vrrp_script check_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 3
    weight -2
    fall 10
    rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 101
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.170
    }
    track_script {
        check_apiserver
    }
}
```

- Create the script to check the API server. **proxy1**
```bash
sudo nano /etc/keepalived/check_apiserver.sh
```
```bash
#!/bin/sh

errorExit() {
    echo "*** $*" 1>&2
    exit 1
}

curl -sfk --max-time 2 https://192.168.1.171:6443/healthz -o /dev/null || errorExit "Error GET https://192.168.1.171:6443/healthz"
curl -sfk --max-time 2 https://192.168.1.172:6443/healthz -o /dev/null || errorExit "Error GET https://192.168.1.172:6443/healthz"
curl -sfk --max-time 2 https://192.168.1.173:6443/healthz -o /dev/null || errorExit "Error GET https://192.168.1.173:6443/healthz"
```

- Make the script executable. **proxy1**
```bash
sudo chmod +x /etc/keepalived/check_apiserver.sh
```

- Configure the haproxy service. **proxy1**
```bash
sudo rm -rf /etc/haproxy/haproxy.cfg
sudo nano /etc/haproxy/haproxy.cfg
```
```conf
global
    log stdout format raw local0
    daemon

defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 1
    timeout http-request    10s
    timeout queue           20s
    timeout connect         5s
    timeout client          35s
    timeout server          35s
    timeout http-keep-alive 10s
    timeout check           10s

frontend apiserver
    bind *:6443
    mode tcp
    option tcplog
    default_backend apiserverbackend

backend apiserverbackend
    option httpchk

    http-check connect ssl
    http-check send meth GET uri /healthz
    http-check expect status 200

    mode tcp
    balance     roundrobin
    
    server node01 192.168.1.171:6443 check verify none
    server node02 192.168.1.172:6443 check verify none
    server node03 192.168.1.173:6443 check verify none
```

- Restart the keepalived and haproxy services. **proxy1**
```bash
sudo systemctl restart keepalived haproxy
```

- Check the status of the keepalived and haproxy services. **proxy1**
```bash
sudo systemctl status keepalived haproxy
```

- Test the reverse proxy. **proxy1**
```bash
nc -v 192.168.1.170 6443
```

### 👉Step 5: Initialize the Kubernetes cluster

```yaml
# cluster01-config.yaml
apiVersion: kubeadm.k8s.io/v1beta3
kind: ClusterConfiguration
metadata:
  name: cluster01
controlPlaneEndpoint: "192.168.1.170:6443"
networking:
  podSubnet: "10.244.0.0/16"
criSocket: "/run/containerd/containerd.sock"
```

- Initialize the Kubernetes cluster on the master node. **node01**
```bash
sudo kubeadm init --config cluster01-config.yaml --upload-certs --v "5"
# Copy the kubeadm join commands for the worker nodes and the master node
```

- Configure the Kubernetes cluster (optional). **node01, node02, node03**
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/
```

- Join the `master` node to the cluster. **node01, node02, node03**
```bash
sudo kubeadm join 192.168.1.170:6443 --token <token> \
        --discovery-token-ca-cert-hash sha256:<hash> \
        --control-plane --certificate-key <key>
```

- Join the `worker` nodes to the cluster. **node04, node05, node06, node07, node08, node09**
```bash
sudo kubeadm join 192.168.1.170:6443 --token <token> \
        --discovery-token-ca-cert-hash sha256:<hash>
```

- Install helm. **node01, node02, node03**
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
sudo ./get_helm.sh
```

- Install the Cilium CNI plugin. **node01**
```bash
# Install the Cilium CLI
helm repo add cilium https://helm.cilium.io/
# Set the Cilium CLI version
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
# Set the Cilium CLI architecture
CLI_ARCH=amd64
# Check the architecture
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
# Download the Cilium CLI
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
# Verify the Cilium CLI
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
# Extract the Cilium CLI
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
# Remove the Cilium CLI
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
# Install the Cilium Helm chart
helm install cilium cilium/cilium --version 1.16.1 \
  --namespace kube-system \
  --set prometheus.enabled=true \
  --set operator.prometheus.enabled=true
```

> **Note:** CNI is a plugin that allows Kubernetes to communicate with the network. Cilium is a CNI plugin that provides networking, security, and observability features.

> CNI = Container Network Interface

- Now wait for all the pots to be `running`. **node01**
```bash
watch kubectl get pods -n kube-system # Press Ctrl+C to exit
```

- Check the status of the nodes. **node01**
```bash
watch kubectl get nodes -o wide # Press Ctrl+C to exit
```

***Congratulations, you have your high availability cluster up and running!***

## 📝Notes

- Interesting commands:
```bash
sudo crictl --runtime-endpoint unix:///run/containerd/containerd.sock ps -a # List all containers

sudo crictl --runtime-endpoint unix:///run/containerd/containerd.sock logs 494942b3f2efb # Get the logs of a container

sudo kubeadm init --cri-socket "/run/containerd/containerd.sock" --pod-network-cidr "10.244.0.0/16" --cluster-name "cluster01" --upload-certs --v=5 # Initialize the cluster without high availability no ip of the reverse proxy for the control plane endpoint 

sudo ufw disable # Disable the firewall
sudo ufw enable # Enable the firewall
sudo ufw allow 22/tcp # Allow SSH

sudo kubeadm reset # Reset the cluster (remove the nodes from the cluster)

kubectl get nodes -o wide # List all nodes with additional information


sudo cat /etc/kubernetes/manifests/kube-apiserver.yaml # Check the API server configuration

sudo tail -f /var/log/nginx/access.log # Check the Nginx access log
sudo tail -f /var/log/nginx/error.log # Check the Nginx error log

sudo apt-get remove -y nginx nginx-common nginx-core # Remove Nginx
sudo apt-get purge -y nginx nginx-common nginx-core # Purge Nginx
sudo apt-get autoremove -y # Remove unused packages

sudo rm -rf $HOME/.kube/config # Remove the Kubernetes configuration
```

## 🔗Links
- 👯 Web hosting company [EliasDH.com](https://eliasdh.com).
- 📫 How to reach us elias.dehondt@outlook.com