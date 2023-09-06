```
eksctl create cluster --name=eksdemo1 \
                      --region=us-east-1 \
                      --zones=us-east-1a,us-east-1b \
                      --without-nodegroup 

```
2. step
   ```
   eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster eksdemo1 \
    --approve

   ```
   3. step
      ```
      eksctl create nodegroup --cluster=eksdemo1 \
                        --region=us-east-1 \
                        --name=eksdemo1-ng-private1 \
                        --node-type=t3.medium \
                        --nodes-min=2 \
                        --nodes-max=4 \
                        --node-volume-size=20 \
                        --ssh-access \
                        --ssh-public-key=kube-demo \
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access \
                        --node-private-networking    
      ```
     ### Pre-requisite-3:  Verify Cluster, Node Groups and configure kubectl cli if not configured
1. EKS Cluster
2. EKS Node Groups in Private Subnets

# Verfy EKS Cluster
```
eksctl get cluster
```
# Verify EKS Node Groups
```
eksctl get nodegroup --cluster=eksdemo1
```
# Verify if any IAM Service Accounts present in EKS Cluster
```
eksctl get iamserviceaccount --cluster=eksdemo1
```
Observation:
1. No k8s Service accounts as of now. 

# Configure kubeconfig for kubectl
eksctl get cluster # TO GET CLUSTER NAME
aws eks --region <region-code> update-kubeconfig --name <cluster_name>
```
aws eks --region us-east-1 update-kubeconfig --name eksdemo1
```

# Verify EKS Nodes in EKS Cluster using kubectl
```
kubectl get nodes
```



# Download IAM Policy
## Download latest
```
curl -o iam_policy_latest.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```


# Create IAM Policy using policy downloaded 
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy_latest.json
```    

- **Important Note:** If you view the policy in the AWS Management Console, you may see warnings for ELB. These can be safely ignored because some of the actions only exist for ELB v2. You do not see warnings for ELB v2.

### Make a note of Policy ARN    
- Make a note of Policy ARN as we are going to use that in next step when creating IAM Role.

# Policy ARN 
Policy ARN:  arn:aws:iam::180789647333:policy/AWSLoadBalancerControllerIAMPolicy


## Step-03: Create an IAM role for the AWS LoadBalancer Controller and attach the role to the Kubernetes service account 
- Applicable only with `eksctl` managed clusters
- This command will create an AWS IAM role 
- This command also will create Kubernetes Service Account in k8s cluster
- In addition, this command will bound IAM Role created and the Kubernetes service account created
### Step-03-01: Create IAM Role using eksctl

# Verify if any existing service account
```
kubectl get sa -n kube-system
```
kubectl get sa aws-load-balancer-controller -n kube-system
```
Obseravation:
1. Nothing with name "aws-load-balancer-controller" should exist

# Replaced name, cluster and policy arn (Policy arn we took note in step-02)
```
eksctl create iamserviceaccount \
  --cluster=eksdemo1 \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::180789647333:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve
```

