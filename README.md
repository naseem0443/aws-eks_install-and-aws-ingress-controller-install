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
      4. HOW TO INSTALL AWS LOAD BALANCER CONTROLLER IN EKS CLUSTER
      1. Download IAM policy for the AWS Load Balancer Controller
         ```
         curl -fsSL -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.0/docs/install/iam_policy.json
         ```
         II. Create an IAM policy called AWSLoadBalancerControllerIAMPolicy
         ```
         aws iam create-policy \
            --policy-name AWSLoadBalancerControllerIAMPolicy \
            --policy-document file://iam-policy.json
         ```
         
         . Create a IAM role and ServiceAccount for the AWS Load Balancer controller using eksctl tool
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
         . Install AWS Load Balancer Controller using Helm
         ```
         helm repo add eks https://aws.github.io/eks-charts
         ```
