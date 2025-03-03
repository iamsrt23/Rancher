# Rancher

**Scenario:**

We Have Managed EKS Application With 4 Worker Nodes 

With Version of 1.28 version  m5 large instances

1—→ 2——> 3——> 4

5—>6—→7—>8

We Have to Upgrade the EKS Cluster to 1.29 version with m5large instances What we have to do is In Rancher 2 Types of Work is there

1. Cordian —> Pause the Node it won’t get jobs
2. Drain —> Drain The Node

FIrstly Cardian the 1 st Node and wait for 2 to 3 min The Job will Go to 1.29 Version Nodes

8 min of Down Time

Similarly Do the remaining Nodes We don’t delete the 1.28 Version Nodes Till No issues in 1.29 Version We have to Put  To avoid Costing

- Max - 0
- Min - 0
- Desired - 1

If any issues occur

- Max - 8
- Min - 4
- Desired -10

**Practicals:**

- We have to create 7 EC2 Instances t2.medium with 20GB add Userdata
    - 3 -master
    - 3 - Worker
    - 1 - Rancher GUI

***userdata.sh***

```python
#!/bin/bash
sudo apt update
sudo curl -LO https://dl.k8s.io/release/v1.32.0/bin/linux/amd64/kubectl
sudo chmod +x kubectl
sudo mv kubectl /usr/local/bin
sudo apt update
sudo curl https://get.docker.com | bash
sudo usermod -a -G docker ubuntu
sudo usermod -a -G root ubuntu
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo reboot
```

cluster.yaml

```python
# https://github.com/rancher/rke/releases/download/v1.6.5/rke_windows-amd64.exe
ssh_key_path: D:\Mine\awskeys\SecOps-Key.pem
cluster_name: rkecluster
kubernetes_version: v1.30.7-rancher1-1
nodes:
  - address: 3.231.215.69
    hostname_override: master01
    #internal_address: 172.16.22.12
    user: ubuntu
    role: [controlplane, worker, etcd]
  - address: 3.231.165.189
    hostname_override: master02
    #internal_address: 172.16.32.37
    user: ubuntu
    role: [controlplane, worker, etcd]
  - address: 44.203.52.85
    hostname_override: master03
    #internal_address: 172.16.42.73
    user: ubuntu
    role: [controlplane, worker, etcd]
  - address: 44.192.116.162
    hostname_override: worker01
    #internal_address: 172.16.42.73
    user: ubuntu
    role: [worker]
  - address: 44.202.188.188
    hostname_override: worker02
    #internal_address: 172.16.42.73
    user: ubuntu
    role: [worker]
  - address: 44.200.201.42
    hostname_override: worker03
    #internal_address: 172.16.42.73
    user: ubuntu
    role: [worker]

authentication:
  strategy: x509
  sans:
    - "LB-RKE-2a2c52e1c77fbbee.elb.us-east-1.amazonaws.com" // DNS NAME
    - "rke.cloudvishwakarma.in" // CREATE RECORD UNDER ROUTE53
services:
  etcd:
    snapshot: true
    creation: 6h
    retention: 24h

# Required for external TLS termination with
# ingress-nginx v0.22+
ingress:
  provider: nginx
  options:
    use-forwarded-headers: "true"

network:
  plugin: flannel
  options:
    flannel_iface: eth0
    flannel_backend_type: vxlan
    flannel_autoscaler_priority_class_name: system-cluster-critical # Available as of RKE v1.2.6+
    flannel_priority_class_name: system-cluster-critical # Available as of RKE v1.2.6+
```

Change the Worker Node and Master Node IP Address accordingly and after that create one Target Group and LoadBalancer select target  only Master and Create Network Loadbalancer

download the rancher.exe file in the same location using below link

```python
https://github.com/rancher/rke/releases/download/v1.6.5/rke_windows-amd64.exe

#to run rke.eke file to deploy k8s Cluster
.\rke.exe up --ignore-docker-version

# After we got kube_config_cluster.yaml
kubectl --kubeconfig=kube_config_cluster.yaml get nodes
kubectl --kubeconfig=kube_config_cluster.yaml get ns
kubectl --kubeconfig=kube_config_cluster.yaml cluster-info

```

**Go to Rancher Instance:**

- install cerbot
    
    ```python
    sudo snap install --classic cerbot
    
    #after installing
    
    cerbot certonly --manual --preferred-challenges=dns --key-type rsa --email rteja170@gmail.com \
    
    							--server https://acme-v02.api.letsencrypt.org/directory --agree-tos -d * .raviteja.live
    ```
    

Goto Route53 create a record 

like - _acme.challengesbjfbjkskjnkkajnj .raviteja.live

Record Type:   Text Type

Enter sample Text Entries

After that it creates 

- cert1.pem
- privkey.oem
- chain1.pem

install docker and Run Rancher Image with certkeys

```python
docker run -d --restart=unless-stopped \
	-p 80:80 -p 443:443 \
	-v /etc/letsencrypt/archive/raviteja.live/cert1.pem:/etc/rancher/ssl/cert.pem \
	-v /etc/letsencrypt/archive/raviteja.live/privkey1.pem:/etc/rancher/ssl/key.pem \
	-v /etc/letsencrypt/archive/raviteja.live/chain1.pem:/etc/rancher/ssl/cacerts.pem \
	--priviliged \
	rancher/rancher:latest
```

TO access through DNS create a Simple Record

[rancer.raviteja.live](http://rancer.raviteja.live) with Ip Address Rancher instance

after open Rancher server

```python
docker logs <rancher-container-id> 2>&1 | grep "Bootstrp Password:"

#o/p is password

```

paste the password in rancher gui

In Rancher GUI

Importing Existing —>  Select generic —>

- Cluster Name:  sba
- Cluster Description: sba

Create

we got some commands copy and paste the command other wise get help from chat gpt 

in Terminal paste the command

```python
kubectl --kubeconfig=kube_config_cluster.yaml create deployment app1 \
      --image kiran2361993/kubegame:v2
      
kubectl --kubeconfig=kube_config_cluster.yaml scale deployment app1 --replicas 10

kubectl --kubeconfig=kube_config_cluster.yaml get pods
```

Play with UI