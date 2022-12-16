
1. Please create the design architecture in the form of a diagram.

https://s3.ap-southeast-1.amazonaws.com/assets.valbury.co.id/oks/Screen+Shot+2022-09-28+at+13.07.06.png




2. In addition, also include brief descriptions on the items below : - List the underlying techstack you believe is required to run the REST API + why - How you would setup the Database replication - Network Subnet design - Security Groups between network
groupings

Tech Stack : 
- Container Runtime : Docker
- Orchestration : Kubernetes / EKS
- Ingress : NGINX Ingress
- Database : MySQL


3. Terraform script for infrastructure provisioning. The script should be organised in such a way that it is Modular, Reusable, Supports Multiple Environments and Easily
Configurable. The script should provision whatever you specify on question number 2.

I put on "terraform" directory


4. Create the deployment pipeline - preferably in Gitlab CI YAML, but Github or Bitbucket
pipeline is also acceptable - to perform the tasks below : - Run the Terraform script
above to provision the underlying infrastructure. - Create a Dockerfile with Alpine base
image and install the application into the container - Deploy the resulting docker to the
targeted instance(s).
