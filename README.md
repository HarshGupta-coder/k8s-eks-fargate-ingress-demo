# k8s-eks-fargate-ingress-demo
A step-by-step guide and configurations for setting up a Kubernetes cluster on AWS Elastic Kubernetes Service (EKS) with Fargate, deploying an application using an Application Load Balancer (ALB) through AWS Load Balancer Controller, and showcasing the power of Kubernetes Ingress for efficient traffic routing.

# Kubernetes Cluster Setup and Demo with AWS EKS

This repository contains instructions and configurations for setting up a Kubernetes cluster on AWS Elastic Kubernetes Service (EKS) using Fargate. The demonstration includes deploying an application with an Application Load Balancer (ALB) using AWS Load Balancer Controller.

## Prerequisites

Ensure you have the following tools installed on your local machine:

- Helm
- eksctl
- AWS CLI
- kubectl

## AWS Configuration

1. **Run `aws configure` on your local terminal and provide AWS Access ID and Secret Access Key.**
   
   This step configures your AWS CLI with the necessary credentials to interact with your AWS account.

## EKS Cluster Setup

2. **Create an EKS cluster using the following command:**

    ```bash
    eksctl create cluster --name demo-cluster --region us-east-1 --fargate
    ```

   This command uses eksctl to create an EKS cluster named 'demo-cluster' in the 'us-east-1' region with Fargate as the compute engine.

   ![image](https://github.com/HarshGupta-coder/k8s-eks-fargate-ingress-demo/assets/54001485/abb77cc2-9602-4dda-81e5-2a8abc9bd029)


4. **Connect kubectl to the EKS cluster:**

    ```bash
    aws eks update-kubeconfig --name demo-cluster --region us-east-1
    ```

   This command configures kubectl to communicate with the newly created EKS cluster.

5. **Create a Fargate profile:**

    ```bash
    eksctl create fargateprofile \
        --cluster demo-cluster \
        --region us-east-1 \
        --name alb-sample-app \
        --namespace game-2048
    ```

   Fargate profiles allow you to run Kubernetes pods without managing the underlying EC2 instances.

   ![image](https://github.com/HarshGupta-coder/k8s-eks-fargate-ingress-demo/assets/54001485/629f1447-43f6-495d-8c92-3ea6b8854a5d)


## Deploying Application with ALB

5. **Apply the configuration for the application, service, and ingress:**

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
    ```

   This step deploys the 2048 game application using the provided YAML configuration. The YAML file defines the deployment, service, and ingress for the application.

6. **Check the status of pods, services, and ingress:**

    ```bash
    kubectl get pods -n game-2048
    kubectl get services -n game-2048
    kubectl get ingress -n game-2048
    ```

   Use these commands to verify that the application pods, services, and ingress resources are running as expected.

   ![image](https://github.com/HarshGupta-coder/k8s-eks-fargate-ingress-demo/assets/54001485/8f233112-f14f-40cc-847f-04354612d311)


## IAM OIDC Connector and ALB Ingress Controller

7. **Associate IAM OIDC provider:**

    ```bash
    eksctl utils associate-iam-oidc-provider --cluster demo-cluster --approve
    ```

   This step associates the IAM OIDC provider with the EKS cluster, allowing it to authenticate Kubernetes service accounts.

8. **Download and create IAM policy:**

    ```bash
    curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

    aws iam create-policy \
        --policy-name AWSLoadBalancerControllerIAMPolicy \
        --policy-document file://iam_policy.json
    ```

    Attach the IAM policy to the service account:

    ```bash
    eksctl create iamserviceaccount \
        --cluster=demo-cluster \
        --namespace=kube-system \
        --name=aws-load-balancer-controller \
        --role-name AmazonEKSLoadBalancerControllerRole \
        --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
        --approve \
        --override-existing-serviceaccounts
    ```

   These steps set up the necessary IAM policies to allow the ALB Ingress Controller to interact with AWS services.

## Installing ALB Ingress Controller with Helm

9. **Install the ALB Ingress Controller using Helm:**

    ```bash
    helm repo add eks https://aws.github.io/eks-charts

    helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
      --set clusterName=demo-cluster \
      --set serviceAccount.create=false \
      --set serviceAccount.name=aws-load-balancer-controller \
      --set region=<region> \
      --set vpcId=<your-vpc-id>
    ```

   Helm is used to deploy the ALB Ingress Controller to the Kubernetes cluster. This command installs the controller with specific configurations.

10. **Verify the ALB Ingress Controller deployment:**

    ```bash
    kubectl get deployment -n kube-system aws-load-balancer-controller
    ```

   Ensure that the ALB Ingress Controller is running with the desired number of replicas.

   ![image](https://github.com/HarshGupta-coder/k8s-eks-fargate-ingress-demo/assets/54001485/56d374f9-64ef-49c9-912f-8be0a8b5ee1a)


11. **Access the application through the ALB in the AWS console:**

    Navigate to the AWS console, go to the EC2 section, and find the Load Balancer. Retrieve the DNS name and access it in the browser to confirm the successful deployment of the application.

    ![image](https://github.com/HarshGupta-coder/k8s-eks-fargate-ingress-demo/assets/54001485/f5b7fe86-a514-4d32-ad40-9f95ca911f29)


Congratulations! Your Kubernetes cluster with Fargate and ALB is now set up, and the application is deployed.

![image](https://github.com/HarshGupta-coder/k8s-eks-fargate-ingress-demo/assets/54001485/98c32192-1b98-4361-949d-e6976e5bc99c)
