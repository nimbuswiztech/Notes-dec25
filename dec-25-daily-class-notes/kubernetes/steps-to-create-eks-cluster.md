# Steps to create EKS cluster

{% stepper %}
{% step %}
### Log in to the AWS Management Console

Open your web browser and navigate to the AWS Management Console. Sign in with your AWS credentials.
{% endstep %}

{% step %}
### Navigate to Amazon EKS

Once you're logged in, go to the AWS EKS console by searching for "EKS" in the AWS services search bar or by selecting "Elastic Kubernetes Service" under the "Containers" section.
{% endstep %}

{% step %}
### Create an EKS Cluster

a. Click the "Create cluster" button to start the cluster creation process.

b. In the "Create EKS cluster" wizard, you'll need to configure several settings:

* Cluster name: Provide a unique name for your EKS cluster.
* Kubernetes version: Select the desired Kubernetes version for your cluster.
* Role for cluster: You need to create or choose an IAM role that EKS can assume to manage resources on your behalf. If you don't have an appropriate role, you can create one during this step.

```
Navigate to IAM service --> Create new role --> choose AWS Service and select usecase dropdown to EKS service -->

Then choose EKS - Cluster --> click on NEXT --> Provide proper name --> click on create 
```

* VPC: Choose an existing Amazon VPC or create a new one for your EKS cluster. Ensure that your VPC has the necessary subnets, security groups, and route tables set up for EKS.
* Subnets: Select the subnets in your VPC where your worker nodes will be launched.
* Endpoint access: Choose whether to allow or deny public access to the Kubernetes API server. This depends on your security requirements.
* Logging: Configure Amazon CloudWatch logging for your EKS cluster if needed.

c. After configuring these settings, click the "Create" button to create your EKS cluster.
{% endstep %}

{% step %}
### Wait for Cluster Creation

EKS will now start provisioning your cluster.

This process may take several minutes to complete. You can monitor the progress on the EKS console.
{% endstep %}

{% step %}
### Once the cluster is active then start creating node group

a. create role for node group

```
Navigate to IAM service --> Create new role --> choose AWS Service and select usecase dropdown to EC2 service -->

Then choose policies --> AmazonEC2ContainerRegistryReadOnly, AmazonEKS_CNI_Policy, AmazonEKSWorkerNodePolicy --> Provide proper name --> Create  
```

b. Create node group

```
Navigte EKS cluster --> click on eks cluster --> navigate node groups --> click Add node group --> provide required info & create node group 
```

Will take few minutes to come up
{% endstep %}

{% step %}
### Connect to EKS cluster

1. Create access key and secret key with user which eks cluster is created with.
2. use `aws configure` and provide information for all the prompts.
3. Execute below command which updates kube config file, which to connect k8s cluster:

```
aws eks update-kubeconfig --region <your_region> --name <your_cluster_name>
```

4. Once kubeconfig file updated, execute command:

```
kubectl get nodes
```

This should show all the nodes in the cluster.
{% endstep %}
{% endstepper %}
