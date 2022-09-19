# Setup AWS managed EKS cluster

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

We need to create Amazon EKS cluster role in the IAM console. [Here](https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html) is the AWS official documentation for Amazon EKS cluster role:

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

After that we need to create a custom VPC. [Here](https://docs.aws.amazon.com/eks/latest/userguide/creating-a-vpc.html) is the AWS official documentation for custom vpc:

1. Open the AWS CloudFormation console at [Here](https://console.aws.amazon.com/cloudformation)

![10](https://user-images.githubusercontent.com/74168188/190577122-faab5074-198f-41a0-bb02-bb58355e5a68.png)

2. From the navigation bar, select an AWS Region that supports Amazon EKS.

3. Choose Create stack, With new resources (standard).

![11](https://user-images.githubusercontent.com/74168188/190577298-ceec082f-e19a-45ee-bd85-8bacb1370e47.png)
Followed
4. Under Prepare template, make sure that Template is ready is selected and then under Template source, select Amazon S3 URL.

5. Paste ```https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml``` URL into the text area under Amazon S3 URL and choose Next:

![12](https://user-images.githubusercontent.com/74168188/190580145-2923aff2-9275-46b9-a91b-34b94acee820.png)

6. On the Specify stack details page, enter the parameters, and then choose Next.

![13](https://user-images.githubusercontent.com/74168188/190580785-408bf0ed-298c-4da6-b353-db6bea224a3a.png)

7. (Optional) On the Configure stack options page, tag your stack resources and then choose Next.

![14](https://user-images.githubusercontent.com/74168188/190581211-824fb385-e443-46c5-b06c-2a75c0483e7a.png)

8. On the Review page, choose Create stack.

![15](https://user-images.githubusercontent.com/74168188/190581466-00fb48df-9dea-416c-9b7d-0e15f4268db6.png)
![16](https://user-images.githubusercontent.com/74168188/190581477-5b44e69a-4faf-4fee-bcd7-17bd86c03894.png)

VPC is created

![17](https://user-images.githubusercontent.com/74168188/190622023-b76a72a9-6df9-4b38-984b-ecb38600654e.png)

Now, To create EKS cluster navigate to the search bar, type EKS, and select Elastic Kubernetes Service

![18](https://user-images.githubusercontent.com/74168188/190082139-6aefde2d-dd88-4efc-ba5e-179b00a78278.png)

Create a new EKS Cluster

![19](https://user-images.githubusercontent.com/74168188/190084760-f9684959-53f1-45ae-bc94-be9b9419965f.png)

Enter cluster name and select cluster service role then click Next

![20](https://user-images.githubusercontent.com/74168188/190085261-8c2e8d2c-de91-4cd2-a444-8d1cf112f9c7.png)

Select VPC, private subnets, and security group

![21](https://user-images.githubusercontent.com/74168188/190622716-8cdbc5ed-429c-46c8-9a0a-6d8047d08020.png)
![22](https://user-images.githubusercontent.com/74168188/190622733-b48e0b96-1507-4832-90cd-30e42a2b43e7.png)

click Next

![23](https://user-images.githubusercontent.com/74168188/190088104-a896b8a9-f018-4df8-8eb5-5e71be4007d7.png)

Review everything then click Create

![24](https://user-images.githubusercontent.com/74168188/190623212-39a20395-0190-4938-9272-fad89cf879af.png)
![25](https://user-images.githubusercontent.com/74168188/190623222-d88f3526-dfe0-40f7-b6d8-66822b66a3e1.png)

Now, we need to add node group to eks cluster. But before this create Node Group role in the IAM console. [Here](https://docs.aws.amazon.com/eks/latest/userguide/create-node-role.html) is the AWS official documentation for Amazon EKS node IAM role:

1. Open the IAM console [Here](https://console.aws.amazon.com/iam/)

![26](https://user-images.githubusercontent.com/74168188/190069626-c53f0210-d3a5-4567-8d1d-386f3c23dfb0.png)

2. Choose Roles, then Create role.

![27](https://user-images.githubusercontent.com/74168188/190069978-c77df6b4-347a-47cc-8196-795eeb355549.png)

3. Under Trusted entity type, select AWS service.

![28](https://user-images.githubusercontent.com/74168188/190070037-5ac52e08-d5c8-4d15-bdd9-47151e440740.png)

4. From the Use cases for other AWS services dropdown list, choose EKS.
5. Choose EKS - Nodegroup for your use case, and then choose Next.

![29](https://user-images.githubusercontent.com/74168188/190090575-b41181a6-a9d4-4cfd-946d-953980aa1c18.png)

6. On the Add permissions tab, choose Next.

![30](https://user-images.githubusercontent.com/74168188/190090968-832f83ca-7a58-4355-8e68-c1201c218f61.png)

7. For Description, enter descriptive text such as Allow EKS to manage nodegroups on your behalf. Then choose Create role.

![31](https://user-images.githubusercontent.com/74168188/190093021-c1a50d89-c051-4c2b-b7c0-a3b3a43aa18f.png)

Let's add node group to cluster. For this navigate to the search bar, type EKS, and select Elastic Kubernetes Service. Select Cluster

![32](https://user-images.githubusercontent.com/74168188/190403805-6115ed7f-9b9d-48c5-b58f-495036786274.png)
![33](https://user-images.githubusercontent.com/74168188/190403957-64cf01a8-138f-4d76-86ee-08c8d057c4b7.png)

In compute unit add a node group

![34](https://user-images.githubusercontent.com/74168188/190626867-5fb9d78c-f261-4af5-981c-65fe9d38f63c.png)

Enter Node Group name and select IAM role

![35](https://user-images.githubusercontent.com/74168188/190627157-0d43bed6-84ff-4e70-a44a-39a10ae35561.png)
![36](https://user-images.githubusercontent.com/74168188/190627166-5bd14dad-bd4b-40a7-a21b-8386a197e1fa.png)


Click Next

![37](https://user-images.githubusercontent.com/74168188/190406344-c440e520-dd7e-43d3-ab10-872cbee8401e.png)
![38](https://user-images.githubusercontent.com/74168188/190406361-a65c59e3-fd87-45ed-a2d9-ac954566a83e.png)

Click Next

Put Node Group in public subnet and click Next

![39](https://user-images.githubusercontent.com/74168188/190407092-06ed66a8-53af-4931-b324-c405ee98edba.png)

Review and click on Create

![40](https://user-images.githubusercontent.com/74168188/190567498-9763890a-d1ba-41fd-834b-5cbca3f15ad4.png)
![41](https://user-images.githubusercontent.com/74168188/190567508-d01f40bb-1c27-41a6-b606-d68ec7f075bc.png)
