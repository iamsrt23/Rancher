# K8s-Upgrade-Zero-DownTime

https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html

What Happen is Every 2-3Months New Version K8s Releases We have to Upgrade that one 

**Prerequisites:**

- Cordon Nodes (Paused )
- Release Notes (1.30 to 1.31)
    - What Update in the new Version
    - Example API ,Ingress Upgrade
- Can’t Downgrade a K8s cluster
    - Test in Lower Environments
- Control Plane of K8s and Nodes On the Same Version
- Cluster Autoscaler
    - Matches with the version of Control Plane
- Five Available Ip Address (Subnet)
- kubelet also match the version of control plane

**Process:**

- EKS doesn’t mean it automatically Upgrade the Worker Nodes It mainly Focus on
    - High Availability of ControlPlane
    - Disaster Recovery
    - Security
    - API
- Upgrade the Node Group/ Nodes / Fargate
    - Managed Node Groups like Desired  Launch Templates
        - RollOut {update node1 and node 2 and node3 like that}
    - Managed by our Own  custom Launch Templates Install Nodes
        - Upgrades Invovlves in Launch Templates or AMI
        - Individually Cordon That Node Unschedule and Upgrade that Node
    - Hybrid (Combination of Both of Them)
- Helm,ArgoCD,Prometheus
    - Test In Lower Environment
- Upgrade the Add-Ons

Create a EKS Cluster 

```python
# Create EKS Cluster
# without node group
eksctl create cluster --name=observability \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup

# Associating IAM with Service account of K8s means k8s access aws services                  
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster observability \
    --approve
   
eksctl create nodegroup --cluster=observability \
                        --region=us-east-1 \
                        --name=observability-ng-private \
                        --node-type=t3.medium \
                        --nodes-min=2 \
                        --nodes-max=3 \
                        --node-volume-size=20 \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking

# Update ./kube/config file
aws eks update-kubeconfig --name observability

# Delete the EKS Cluster 
eksctl delete cluster --name observability
```

**Goto EKS DashBoard:**

```python

# Upgrade the ControlPlane
eksctl upgrade cluster --name <cluster-name> --region us-east-1 --version 1.31 --approve

# Time Take to 30mins

```

**Go to Compute:**

- We can have update now Option
    - Update Strategy : Rolling Update
- Do Create A new Node Group with Node Created With Latest Version uninstall old one

**GoTo Add-Ons:**

- Update to the Latest Version

**How to Test The Upgrade:**

- Execute Functional Test
    - Helm,ArgoCD