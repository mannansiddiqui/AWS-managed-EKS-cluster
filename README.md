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

To setup EKS cluster we have followed [this](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html) AWS official documentation.

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

4. Under Use case, choose EC2 then Choose Next.

![29](https://user-images.githubusercontent.com/74168188/190968676-9359495d-5e32-490c-8bdf-40b9303ebf99.png)

5. On the Add permissions tab, choose AmazonEKSWorkerNodePolicy, AmazonEC2ContainerRegistryReadOnly, and AmazonEKS_CNI_Policy then choose Next.

![30](https://user-images.githubusercontent.com/74168188/190970077-5e540ca7-9380-460c-b353-0df4e2f393c9.png)
![31](https://user-images.githubusercontent.com/74168188/190970088-0283648b-35bd-4a3b-94ae-7f2b71e5a2c9.png)
![32](https://user-images.githubusercontent.com/74168188/190970098-1bef09a5-b5f9-43e5-b22b-4dea035c90db.png)

6. On the Name, review, and create page enter a unique name for your role, such as AmazonEKSNodeRole and replace the current text with descriptive text such as Amazon EKS - Node role.

![33](https://user-images.githubusercontent.com/74168188/190970898-5b905f6a-2a78-43cb-96e0-75f0202ffb0d.png)
![34](https://user-images.githubusercontent.com/74168188/190970910-647a322a-8411-4b93-a056-6c372736263c.png)

Then choose Create role.

![35](https://user-images.githubusercontent.com/74168188/190971663-1c3dc450-6696-4387-80de-0a17949adfe2.png)

Let's add node group to cluster. For this navigate to the search bar, type EKS, and select Elastic Kubernetes Service. Select Cluster

![36](https://user-images.githubusercontent.com/74168188/190403805-6115ed7f-9b9d-48c5-b58f-495036786274.png)
![37](https://user-images.githubusercontent.com/74168188/190403957-64cf01a8-138f-4d76-86ee-08c8d057c4b7.png)

In compute unit add a node group

![38](https://user-images.githubusercontent.com/74168188/190626867-5fb9d78c-f261-4af5-981c-65fe9d38f63c.png)

Enter Node Group name and select IAM role

![39](https://user-images.githubusercontent.com/74168188/190972237-8796eec2-6b6d-46ba-91eb-54be476017d0.png)
![40](https://user-images.githubusercontent.com/74168188/190972253-dd797a94-3ddd-4d05-accf-69dc194748f9.png)

Click Next

![41](https://user-images.githubusercontent.com/74168188/190406344-c440e520-dd7e-43d3-ab10-872cbee8401e.png)
![42](https://user-images.githubusercontent.com/74168188/190406361-a65c59e3-fd87-45ed-a2d9-ac954566a83e.png)

Click Next and put Node Group in public subnet and click Next

![43](https://user-images.githubusercontent.com/74168188/190973054-6751edfe-497a-44b4-8b9c-2bee9b7bf1ac.png)

Review and click on Create

![44](https://user-images.githubusercontent.com/74168188/190973540-7cdc4e37-82aa-478d-993c-e4ff395b7eee.png)
![45](https://user-images.githubusercontent.com/74168188/190973548-b85e9feb-e755-4183-8103-6e7955d9c7dc.png)

Node group is attached successfully

![46](https://user-images.githubusercontent.com/74168188/190975735-b749babc-626d-4cb1-9d1a-3f9597002467.png)

To access this eks cluster we need a bastion server. To launch a bastion server, navigate to the search bar, type EC2, and select EC2.

![47](https://user-images.githubusercontent.com/74168188/190962213-95a4d2d7-f964-4042-9b05-6cafe81decb4.png)

Now, Select Launch instance

![48](https://user-images.githubusercontent.com/74168188/190962347-b6bb6af8-6cd4-43d6-a00b-15a1f4cbc352.png)

Enter few details like Instance name, AMI, Instance type, key pair to attach with instance and VPC then click Launch instance.

![49](https://user-images.githubusercontent.com/74168188/190976414-b9186e0b-7aa3-4aeb-8f37-134b45f8c345.png)
![50](https://user-images.githubusercontent.com/74168188/190976425-8219a044-ffe1-43eb-93bc-36bbabd4a73c.png)

Bastion server is successfully launched

![51](https://user-images.githubusercontent.com/74168188/190977292-d99d22f4-e1b7-4831-8bc6-e4b77f5fd047.png)

Now, ssh to bastion server from local

![52](https://user-images.githubusercontent.com/74168188/190978252-18cda3de-3e4d-4311-9826-af68b002f236.png)

Install aws cli so that we can run aws configure command and get the kubeconfig file in bastion server. [Here](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) is the AWS official documentation to install AWS CLI.

Run these commands to install aws cli:

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
![53](https://user-images.githubusercontent.com/74168188/190980313-b8bbb2ba-c31e-41d0-bbe0-97330ed737df.png)
![54](https://user-images.githubusercontent.com/74168188/190980425-630ad6af-3e28-471c-88eb-754dfd486d1c.png)

Now run ```aws configure``` and enter access key and secret key which is used to create a EKS cluster else it may not work. You can get keys from security credentials of AWS management console or create new by clicking Create New Access Key

![55](https://user-images.githubusercontent.com/74168188/190984767-662c78b1-4163-4859-8621-a832b0843383.png)
![56](https://user-images.githubusercontent.com/74168188/190984998-8d03237f-4ca7-45d1-a144-b5b9498110cd.png)

![57](https://user-images.githubusercontent.com/74168188/191001861-3e267537-6da1-4c2c-9e72-d9cd1195c91c.png)

AWS cli is successfully installed and configured.

Now we also need kubectl installed on bastion server to control EKS cluster. [Here](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html) is the AWS official documentation to install kubectl.

Run these commands to install kubectl:
```
curl -o kubectl https://s3.us-west-2.amazonaws.com/amazon-eks/1.22.6/2022-03-09/bin/linux/amd64/kubectl
chmod +x ./kubectl
mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
```

![58](https://user-images.githubusercontent.com/74168188/191004190-97dd0c81-c41c-43df-a616-830444ebb1b5.png)

kubectl is successfully installed.

Now we need a kube config file. Run ```aws eks update-kubeconfig --region <region-code> --name <my-cluster>``` on the bastion server. It will download the kube config file and store in as **.kube/config** file

![59](https://user-images.githubusercontent.com/74168188/191004027-8ab7ee31-77e9-45af-9b00-9e19b2b39af6.png)

Now we need to allow 443 port in security group of worker nodes as kubectl work on 443 port.

![60](https://user-images.githubusercontent.com/74168188/191012611-c2a2e0a4-30a0-4afc-991b-fa491a7065a2.png)

We are able to do **kubectl get all** it means everything is configured correctly.

![61](https://user-images.githubusercontent.com/74168188/191015929-5178fc25-012b-4b7c-9bff-5a9b6ef24915.png)

#### Step-2: Create NodeJS deployment and service

To create a sample NodeJS app we used [this](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/) nodejs official documentation .

Firstly we need to create a **package.json** file that describes your app and its dependencies:
```
{
  "name": "docker_web_app",
  "version": "1.0.0",
  "description": "Node.js on Docker",
  "author": "First Last <first.last@example.com>",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.16.1"
  }
}
```

![62](https://user-images.githubusercontent.com/74168188/191017233-b1b29ffb-5600-42e9-880a-d05dd9f7dc41.png)

Then, create a server.js file that defines a web app using the Express.js framework.

```
'use strict';

const express = require('express');

// Constants
const PORT = 80;
const HOST = '0.0.0.0';

// App
const app = express();
app.get('/', (req, res) => {
  res.send('Hello World');
});

app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```

![63](https://user-images.githubusercontent.com/74168188/191020622-9410d59e-68bd-4d3c-937c-9e9b35deddd6.png)

Now create a Dockerfile and build a docker image using node as base image then push to docker hub.

Create file using ```sudo vim Dockerfile``` and put this:

```
FROM node:16

# Create app directory
WORKDIR /usr/src/app

# Install app dependencies
# A wildcard is used to ensure both package.json AND package-lock.json are copied
# where available (npm@5+)
COPY package*.json ./

RUN npm install
# If you are building your code for production
# RUN npm ci --only=production

# Bundle app source
COPY . .

EXPOSE 80
CMD [ "node", "server.js" ]
```

![64](https://user-images.githubusercontent.com/74168188/191021891-98fc554b-8a72-4625-b816-b3a2c22dedc3.png)

Create a .dockerignore file in the same directory as your Dockerfile with following content:
```
*
!package*.json
!Dockerfile
!server.js
```

![65](https://user-images.githubusercontent.com/74168188/191188533-d5fcf73f-7f2b-4e44-8707-890283e1200a.png)

But to build docker image we need to install docker in bastion server. To install Docker in Amazon Machine Linux 2 image use ```sudo yum install docker -y```

![66](https://user-images.githubusercontent.com/74168188/191027590-ac0e5e3f-2420-47f0-8068-7ffb50843e8a.png)
![67](https://user-images.githubusercontent.com/74168188/191027608-82699959-4eec-41ea-b334-caee4b04c310.png)

Now start and enable docker using:

```
sudo systemctl start docker
sudo systemctl enable docker
```

![68](https://user-images.githubusercontent.com/74168188/191028327-758ded41-1748-47b6-9c4c-1489fa71ac3a.png)

Build a docker image using ```sudo docker build -t <username>/<image_name>:<version> <path_of_docker_file>```

![69](https://user-images.githubusercontent.com/74168188/191183666-6dc78ae3-b355-4e30-b870-9f404b1cd4e6.png)

Docker image is successfully created and can be seen using ```sudo docker image ls```

![70](https://user-images.githubusercontent.com/74168188/191184340-5c427e84-e524-4926-96cd-faaf1c50d5fa.png)

To push this image to docker hub you need to login inside dockerhub from bastion server using ```sudo docker login``` and enter username and password.

![71](https://user-images.githubusercontent.com/74168188/191184705-89f953c7-3938-401f-8adf-e656b502b81e.png)

Now to push this image run ```sudo docker push mannansiddiqui/nodejs-app:v1```

![72](https://user-images.githubusercontent.com/74168188/191187299-cb223a35-de6f-4a25-bad5-f2bbc8e3966c.png)

Image is successfully pushed to docker hub.

![73](https://user-images.githubusercontent.com/74168188/191187923-445a48e7-07a5-4301-81a7-7fca487a7f5d.png)

Now time to create a Nodejs deployment manifest file. But before this create a storage class and PV so that this PV can be attached to nodejs-app container.

Create a storage class using ```sudo vim sc.yaml``` and put this:
```
