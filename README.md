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

Create a new EKS Cluster
![16](https://user-images.githubusercontent.com/74168188/190084760-f9684959-53f1-45ae-bc94-be9b9419965f.png)

Enter cluster name and select cluster service role then click Next
![17](https://user-images.githubusercontent.com/74168188/190085261-8c2e8d2c-de91-4cd2-a444-8d1cf112f9c7.png)

Select VPC, private subnets, and security group
![18](https://user-images.githubusercontent.com/74168188/190087916-d7d0887d-024a-4cc0-a45e-de6febff8c16.png)
![19](https://user-images.githubusercontent.com/74168188/190087929-2f398cf4-27f3-4654-8ccb-751ca3fcd2a8.png)

click Next
![20](https://user-images.githubusercontent.com/74168188/190088104-a896b8a9-f018-4df8-8eb5-5e71be4007d7.png)

Review everything then click Create
![21](https://user-images.githubusercontent.com/74168188/190088427-a6bc4ddd-3dac-4896-a44b-4e35fd1eb545.png)
![22](https://user-images.githubusercontent.com/74168188/190088438-f1a68e96-ee1b-4bbe-bdcb-1b612ec19378.png)

Now, we need to add node group to eks cluster. But before this create Node Group role in the IAM console:

1. Open the IAM console [Here](https://console.aws.amazon.com/iam/)
![23](https://user-images.githubusercontent.com/74168188/190069626-c53f0210-d3a5-4567-8d1d-386f3c23dfb0.png)

2. Choose Roles, then Create role.
![24](https://user-images.githubusercontent.com/74168188/190069978-c77df6b4-347a-47cc-8196-795eeb355549.png)

3. Under Trusted entity type, select AWS service.
![25](https://user-images.githubusercontent.com/74168188/190070037-5ac52e08-d5c8-4d15-bdd9-47151e440740.png)

4. From the Use cases for other AWS services dropdown list, choose EKS.
5. Choose EKS - Nodegroup for your use case, and then choose Next.
![26](https://user-images.githubusercontent.com/74168188/190090575-b41181a6-a9d4-4cfd-946d-953980aa1c18.png)

6. On the Add permissions tab, choose Next.
![27](https://user-images.githubusercontent.com/74168188/190090968-832f83ca-7a58-4355-8e68-c1201c218f61.png)

7. For Description, enter descriptive text such as Allow EKS to manage nodegroups on your behalf. Then choose Create role.
![28](https://user-images.githubusercontent.com/74168188/190093021-c1a50d89-c051-4c2b-b7c0-a3b3a43aa18f.png)
