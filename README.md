# Steup AWS managed EKS cluster

#### Description:

- Setup EKS cluster with custom VPC
- Create NodeJS deployment and service
- PV & storage class
- Create ingress controller and access above created deployment using ingress and point it to the domain (application should be accessible through a browser) 
- HPA & probs
- SSL certificate
- Kustomization
- Jenkins declarative pipeline to deploy NodeJS app on K8s cluster
- Use database and connect it with frontend
- Configure monitoring using prometheus and grafana

#### Step-1: Setup EKS cluster with custom VPC

Firstly, Log in to the AWS management console.
![1](https://user-images.githubusercontent.com/74168188/178555843-f062573f-166c-4b06-b947-d2d11da46507.png)

We need to create Amazon EKS cluster role in the IAM console:

1. Open the IAM console [Here](https://console.aws.amazon.com/iam/)
![2](https://user-images.githubusercontent.com/74168188/190069626-c53f0210-d3a5-4567-8d1d-386f3c23dfb0.png)

2. Choose Roles, then Create role.
![3](https://user-images.githubusercontent.com/74168188/190069978-c77df6b4-347a-47cc-8196-795eeb355549.png)

3. Under Trusted entity type, select AWS service.
![4](https://user-images.githubusercontent.com/74168188/190070037-5ac52e08-d5c8-4d15-bdd9-47151e440740.png)

4. From the Use cases for other AWS services dropdown list, choose EKS.
5. Choose EKS - Cluster for your use case, and then choose Next.
![5](https://user-images.githubusercontent.com/74168188/190070109-45d85743-e089-4115-8187-92e3b6e9b7fd.png)

6. On the Add permissions tab, choose Next.
![6](https://user-images.githubusercontent.com/74168188/190070600-b218672a-84b3-4317-9052-70616a74f59f.png)

7. For Role name, enter a unique name for your role, such as eksClusterRole.
8. For Description, enter descriptive text such as Amazon EKS - Cluster role.
![7](https://user-images.githubusercontent.com/74168188/190070704-f9472b3b-ce40-4069-b6c8-3f48d85999a6.png)

9. Choose Create role.
![8](https://user-images.githubusercontent.com/74168188/190070737-1a187b57-293f-4f78-a79e-31b03a033425.png)

After login, navigate to the search bar, type EKS, and select Elastic Kubernetes Service
