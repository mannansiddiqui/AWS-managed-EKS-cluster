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

Now time to create a Nodejs deployment manifest file. But before this create a storage class and PV so that this PV can be attached to nodejs container. [Here](https://kubernetes.io/docs/concepts/storage/storage-classes/) is the Kubernetes official documentation to create storage class. [Here](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) is the Kubernetes official documentation to create PVC.

Create a storage class using ```sudo vim sc.yml``` and put this:
```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mysc
parameters:
  fsType: ext4
  type: gp2
provisioner: kubernetes.io/aws-ebs
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: failure-domain.beta.kubernetes.io/zone
    values:
    - ap-south-1b
    - ap-south-1a
# mountOptions:
# - debug
```

![74](https://user-images.githubusercontent.com/74168188/191201717-9abf552e-616a-48e4-87a6-9cd8f4baf4b5.png)

To create this storage class use ```kubectl create -f sc.yml```

![75](https://user-images.githubusercontent.com/74168188/191202401-774e494c-88f9-41eb-bfaf-fca2ef5b5716.png)

Create a PVC so that PV is dynamically created. To create a PVC manifest file use ```sudo vim pvc.yml``` and put this:

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc
spec:
  accessModes:
  - ReadWriteOnce
  volumeMode: Filesystem
  storageClassName: "mysc"
  resources:
    requests:
      storage: 1Gi
```

![76](https://user-images.githubusercontent.com/74168188/191203738-591736e5-0824-4d2f-be02-27453df311ba.png)

To create a PV from storage class create PVC run ```kubectl create -f pvc.yml```

![77](https://user-images.githubusercontent.com/74168188/191204154-3c5bce09-b107-4659-8278-e26f2a7050b5.png)

Check storage class, pvc, and pv is created or not using ```kubectl get sc```, ```kubectl get pvc``` and ```kubectl get pv```

![78](https://user-images.githubusercontent.com/74168188/191204652-088ac3a2-30db-4cf2-9c2d-0f213be86dde.png)

As seen in above screenshot PV is not created because storage class will only create a PV when there is consumer available i.e. pod.

Now create a deployment for nodejs app. Create a deployment file using ```sudo vim deployment.yml```. [Here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) is the Kubernetes official documentation to create a deployment.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mydeployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodejspod
  template:
    metadata:
      labels:
        app: nodejspod
    spec:
      volumes:
      - name: pvfromebs
        persistentVolumeClaim:
          claimName: mypvc
      containers:
      - name: nodejscontainer
        image: mannansiddiqui/nodejs-app:v1
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /var
          name: pvfromebs
```

![79](https://user-images.githubusercontent.com/74168188/191206986-2b94c311-f614-4bc2-b590-8c18f2dca0da.png)

To create a deployment run ```kubectl create -f deployment.yml```

![80](https://user-images.githubusercontent.com/74168188/191206504-ff3076fa-4cac-4076-bee8-62dfb4873a78.png)

Now check all i.e. storage class, pvc, pv, deployment, and pods.

![81](https://user-images.githubusercontent.com/74168188/191207467-60c74ede-2f33-4730-86ff-c802696ac9ba.png)

Time to expose this nodejs app to outside this cluster. For this create a service manifest file using ```sudo vim service.yml```. [Here](https://kubernetes.io/docs/concepts/services-networking/service/) is the Kubernetes official documentation to create service.

```
apiVersion: v1
kind: Service
metadata:
  name: mysvc
spec:
  type: LoadBalancer
  selector:
    app: nodejspod
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

![82](https://user-images.githubusercontent.com/74168188/191209900-a8f8d47d-7285-4e34-b335-df3fb82d215f.png)

To create service with LoadBalancer type run ```kubectl create -f service.yml```

![83](https://user-images.githubusercontent.com/74168188/191210387-9e9264f0-67be-4817-8905-2fc041bdde19.png)

Check service is succesfully created or not using ```kubectl get svc```

![84](https://user-images.githubusercontent.com/74168188/191227626-b5a95f01-530c-4758-9e00-2b1d416cf6dc.png)

Now try to hit External IP.

![85](https://user-images.githubusercontent.com/74168188/191227883-a0ab63b8-8078-459f-8059-09dd77a70894.png)

Site is working fine.

#### Step-3: PV & storage class

Storage class and PV is created in starting of Step-2.

#### Step-4: Create ingress controller and access above created deployment using ingress and point it to the domain (application should be accessible through a browser)

In order for the Ingress resource to work, the cluster must have an ingress controller running. Kubernetes as a project supports and maintains AWS, GCE, and nginx ingress controllers. We will use Nginx ingress controller. [Here](https://kubernetes.github.io/ingress-nginx/deploy/) is the official documentation to install Nginx ingress controller.

In AWS, we use a Network load balancer (NLB) to expose the NGINX Ingress controller behind a Service of Type=LoadBalancer. So flow will be like:

```User----->LoadBalancer----->Ingress----->Service----->Pod```

Run ```kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.3.1/deploy/static/provider/aws/deploy.yaml``` to install ingress controller.

![86](https://user-images.githubusercontent.com/74168188/191447921-8994acb4-a456-483c-a5cb-6904ed853a47.png)

To check all ingress-nginx-controller pods and service are running run ```kubectl get all -n ingress-nginx``` as all nginx ingress controller pods and services are running inside **ingress-nginx** namespace.

![87](https://user-images.githubusercontent.com/74168188/191667523-10421d21-3b8b-4195-95e6-605073fe69cc.png)

Now time to create ingress resource. [Here](https://kubernetes.io/docs/concepts/services-networking/ingress/) is the official documentation to create ingress resource. We need an domain for ingress resource.

Purchase a free domain from freenom.com. For this, search any domain and click select, then checkout and complete the order.

![88](https://user-images.githubusercontent.com/74168188/178556964-db9c96e3-443e-4be7-88d1-0b450c5feb7b.png)
![89](https://user-images.githubusercontent.com/74168188/178556984-c7d8378f-e7b4-452b-a4f1-3a24d90cd58b.png)
![90](https://user-images.githubusercontent.com/74168188/178557005-4c881e54-da28-4ab4-8c7b-5b28939de62e.png)

Now, select My Domains inside Services.

![91](https://user-images.githubusercontent.com/74168188/178632468-d7ba16f6-37db-441a-8376-3aef7e3f6296.png)

Click the manage button of the domain that you want to configure.

![92](https://user-images.githubusercontent.com/74168188/178655181-8b5d6421-d8b6-4f6c-a862-3ff0e7b3df05.png)

Under management tools select nameservers. Here we need to provide the nameserver that will convert domain name to loadbalancer. Nameservers we can get by createing a hosted zone using AWS Route53. 

Now, create a hosted zone using the Route53 service of AWS and map above domain to LoadBalancer.

![93](https://user-images.githubusercontent.com/74168188/178700658-ba293416-af6e-4deb-aae0-9a328b9acd1c.png)

Fill in the domain name you want to route traffic, then click create a hosted zone.

![94](https://user-images.githubusercontent.com/74168188/178701549-a95ca56b-1f4b-40d0-8a3a-775267ee18da.png)

![95](https://user-images.githubusercontent.com/74168188/178701591-13e43594-3576-480f-98f8-93d6647fc3cb.png)

A hosted zone is now created. Create a record to route traffic to the LoadBalancer.

![96](https://user-images.githubusercontent.com/74168188/191676903-0e703aec-f3cc-41de-9577-c0ee6b3b318a.png)
![97](https://user-images.githubusercontent.com/74168188/191677723-2da0d0bb-9327-4040-bc03-14bb8663246d.png)
![98](https://user-images.githubusercontent.com/74168188/191677920-22d91602-d522-4718-bba8-2a8d4b5bb394.png)

Also, copy the nameserver from records of hosted zone and paste it in freenom.

![99](https://user-images.githubusercontent.com/74168188/191744112-6e236dc9-abd7-465d-afea-6778cf5321fd.png)
![100](https://user-images.githubusercontent.com/74168188/191744184-62798fa3-5dbc-49b7-9c3e-3a80c6f8b9c6.png)

Now, Create a manifest file for ingress resource using ```sudo vim ingress.yml``` and put this:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: nodejs.mannan18.ml
    http:
        paths:
        - pathType: Prefix
          path: "/nodejs"
          backend:
            service:
              name: mysvc
              port:
                number: 80
```

![99](https://user-images.githubusercontent.com/74168188/191509427-5dfa6274-fd87-4894-ba93-49bafa806d7d.png)

To create ingress resource run ```kubectl apply -f ingress.yml``` from bastion server.

![100](https://user-images.githubusercontent.com/74168188/191510378-e0700ff3-7949-4290-994c-6ae348e896e0.png)

Now, to check ingress is created is not run ```kubectl get ingress``` and check address is assigned or not.

![101](https://user-images.githubusercontent.com/74168188/191669303-95e0e981-fa75-4321-8826-99e45812a7ce.png)

Let's test ingress is working or not. First hit the ***nodejs.mannan18.ml*** in this case we will not get any result because rule is not created for ***/*** route also default backend is not created. 

![102](https://user-images.githubusercontent.com/74168188/191725519-3413c431-f7e9-4bb6-9908-96a59d0d894d.png)

After this hit ***nodejs.mannan18.ml/nodejs*** in this case we should get result as we have created rule for ***/nodejs*** let's see...

![103](https://user-images.githubusercontent.com/74168188/191725554-a86627a0-a0f3-4a26-b218-95b65ac79e49.png)

Nodejs app is accessible...
