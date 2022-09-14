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
![2](https://user-images.githubusercontent.com/74168188/190071634-bf417dd2-5e8b-4342-b18f-57973ddff4b5.png)

We need to create Amazon EKS cluster role in the IAM console:

1. Open the IAM console [Here](https://console.aws.amazon.com/iam/)
![3](https://user-images.githubusercontent.com/74168188/190069626-c53f0210-d3a5-4567-8d1d-386f3c23dfb0.png)

2. Choose Roles, then Create role.
![4](https://user-images.githubusercontent.com/74168188/190069978-c77df6b4-347a-47cc-8196-795eeb355549.png)

3. Under Trusted entity type, select AWS service.
![5](https://user-images.githubusercontent.com/74168188/190070037-5ac52e08-d5c8-4d15-bdd9-47151e440740.png)

4. From the Use cases for other AWS services dropdown list, choose EKS.
5. Choose EKS - Cluster for your use case, and then choose Next.
![6](https://user-images.githubusercontent.com/74168188/190070109-45d85743-e089-4115-8187-92e3b6e9b7fd.png)

6. On the Add permissions tab, choose Next.
![7](https://user-images.githubusercontent.com/74168188/190070600-b218672a-84b3-4317-9052-70616a74f59f.png)

7. For Role name, enter a unique name for your role, such as eksClusterRole.
8. For Description, enter descriptive text such as Amazon EKS - Cluster role.
![8](https://user-images.githubusercontent.com/74168188/190070704-f9472b3b-ce40-4069-b6c8-3f48d85999a6.png)

9. Choose Create role.
![9](https://user-images.githubusercontent.com/74168188/190070737-1a187b57-293f-4f78-a79e-31b03a033425.png)

After that we need to create a custom VPC. For this search VPC in search bar of AWS console.

![10](https://user-images.githubusercontent.com/74168188/190072040-926bfe5a-3b14-4f1a-8a41-b3ca2d28a1b5.png)

Then click on **Create VPC**
![11](https://user-images.githubusercontent.com/74168188/190072158-f5aa51e9-bbcd-4c6e-8fa0-8ac9b340e19e.png)

Enter few details and then click **Create VPC**
![12](https://user-images.githubusercontent.com/74168188/190081615-fa824e10-6cce-4d50-9607-207152e7d0b4.png)
![13](https://user-images.githubusercontent.com/74168188/190081628-4a77589a-0540-4e61-84df-8b06ddb848ce.png)
![14](https://user-images.githubusercontent.com/74168188/190081826-58654262-773f-477f-a596-fbd021de3368.png)

Now, navigate to the search bar, type EKS, and select Elastic Kubernetes Service
![15](https://user-images.githubusercontent.com/74168188/190082139-6aefde2d-dd88-4efc-ba5e-179b00a78278.png)
