## DOCUMENTATION FOR ANSIBLE-AUTOMATE PROJECT

In this project, we will be working on automating routine IT operations with [Ansible Configuration Management](https://www.redhat.com/en/topics/automation/what-is-configuration-management#:~:text=Configuration%20management%20is%20a%20process,in%20a%20desired%2C%20consistent%20state.&text=Managing%20IT%20system%20configurations%20involves,building%20and%20maintaining%20those%20systems.). The project will show how deploying Ansible will help to simplify complex tasks and streamline IT infastructure. We will also work with Jenkins to configure and execute build jobs whilst writing code using declarative language such as **`YAML`**.
 

### <br>Introduction to Ansible Configuration Management<br/>

The goal of this project is to demonstrate Ansible's powerful automation capabilities. Ansible is an open source, command line software application that is used for automating IT operations such as deploying applications, managing configurations scaling infrastructure and other activities involving many repetitive tasks. Ansible's main strengths are simplicity and ease of use. It lets IT professionals quickly and easily deploy multi tier apps. Rather than needing to write lengthy code to automate our systems, we simply list the tasks that require automation by writing a Playbook and Ansible figures out how to get our systems to the state we need them to be in. It is also important to note Ansible has a strong focus on security and reliability and as such it has very minimal moving parts. A great exapmle of this is that Ansible is agentless. This means that the devices or infrastructure it monitors do not require any proprietary software agent to be installed on them beforehand. Ansible has two major types of files: 

**1.** The Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate.
   
**2.** Ansible Playbooks are lists of tasks that are automatically executed for the specified inventory or groups of hosts. One or more Ansible tasks can be combined to make a play — an ordered grouping of tasks mapped to specific hosts—and tasks are executed in the order in which they are written.

![0_sMSfIbPO8mH299to](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4134dac2-80be-4957-bdf5-9357889f51f5)

#### Ansible as a Jump Server

A [Jump Server](https://en.wikipedia.org/wiki/Jump_server) (sometimes also referred as [Bastion Host](https://en.wikipedia.org/wiki/Bastion_host)) is an intermediary server through which access to internal network can be provided. If you think about the current architecture we are working on, ideally, the webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provides better security and reduces [attack surface](https://en.wikipedia.org/wiki/Attack_surface).

On the diagram below, the Virtual Private Network (VPC) is divided into [two subnets](https://docs.aws.amazon.com/vpc/latest/userguide/how-it-works.html) – Public subnet has public IP addresses and Private subnet is only reachable by private IP addresses.

![bastion host](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7edfe278-2217-45c4-b35d-ac05bfd8ff47)

Further on in our future projects, we will implement a proper Bastion Host. But for now, we will make do with developing Ansible scripts to simulate the use of a Jump Box/Bastion Host to access our Web Servers. Therefore, based on the infrastructure to be used and the goal of this project, it shall consist of three parts:

**1.** Install and Set-up Jenkins on EC2 Instance.

**2.** Install and Configure Ansible to act as a Jump Server/Bastion Host.

**3.** Create a Simple Ansible Playbook to Automate Server Configuration.

### <br>Install and Set-up Jenkins on EC2 Instance<br/>

To begin our project we need to install and set up Jenkins. Jenkins is an open-source automation tool written in Java with plugins built for continuous integration. Jenkins is used to build and test software projects continuously making it easier for developers to integrate changes to the project, and making it easier for users to obtain a fresh build. It also allows developers to continuously deliver software by integrating with a large number of testing and deployment technologies. We deploy Jenkins by implementing the following steps:

#### <br>Step 1: Provision EC2 Instance<br/>

We begin by spinning up an EC2 Instance of Ubuntu Server: We launch our EC2 instance by following [these steps:](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-launch-instance) 

**i.** We open the AWS console and click on **"EC2"**, then we scroll up and click on **"Launch Instance"**.

![launch EC2 instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d331142c-a425-485d-9338-5e8f21d2a37d)

**ii.** Under **Name and tags**, we provide a unique name for our server.

![name and tags](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/e9c2dc00-0a59-433a-ab2d-6136183f190f)
  
**iii.** From the **Applications and Amazon Machine Image (AMI Image)** tab, we ensure we select the free tier eligible version of Ubuntu Linux Server 22.04 LTS (HVM).

![Application and OS Image](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ae844641-2121-49de-99e3-67c8621c4027)

**iv.** Under **Key pair**, we select an existing one. (You can create a new key pair if you do not have one and the same key pair can be used for all the instances that will be provisioned in this project.)

![Key Pair](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/65facdd1-4be3-4ec5-aac4-aadd74821653)
  
**v.** And then finally, we click on **"Launch Instance"**
  
![Launch Instance](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ed623db9-831f-4c86-bc46-f0e7201c18f6)

#### <br>Step 2: Open Port 8080<br/>

**Jenkins** uses TCP Port 8080 by default and as this will be our first installation, we will therefore need to open Port 8080 to allow traffic from anywhere. To implement this, we need to add a rule to the Security Group of our Web Server:

**i.** In the AWS  console navigation pane, we choose **Instances**.

![Instances](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/75d2208f-d030-4f44-9667-23521332607f)

**ii.** We click on our Instance ID to get the details of our EC2 instance and in the bottom half of the screen, we choose the **Security** tab. **Security groups** lists the security groups that are associated with the instance. Inbound rules displays a list of the **inbound rules** that are in effect for the instance.

![instance summary](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ee3841b0-95f1-43be-94b1-6210d7bea296)

**iii.** For the security group to which we will add the new rule, we choose the security group ID link to open the security group.

![security groups](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/f4453010-cf80-4e64-aab5-d6ac89c2a5fc)

**iv.** On the **Inbound rules** tab, we choose **Edit inbound rules**.

![Edit Inbound Rules](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ca7e7378-eba1-455e-a439-f91dd34cc038)

**v.** On the **Edit inbound rules** page, we do the following:

+ Choose **Add rule**.

+ For **Port Range**, enter **8080** 

+ In the space with the magnifying glass under **Source**, choose **Anywhere**.

+ Click on **Save rules** at the bottom right corner of the page.

![open port 8080](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d41b3f43-59c9-4c78-8a80-0fa40515399a)

#### <br>Step 3: Create and Allocate Elastic IP Address to Jenkins-Ansible Server<br/>

Considering that we'll be using Jenkins with Github and configuring Web Hooks in this project, it will make our job easier to create and allocate an elastic IP adress to our Jenkins-Ansible Server. This is beacuse everytime we stop/start the server, there will be a need to keep reconfiguring Github Web Hooks to a new IP address. Having an elastic IP address (which will not change when we stop/start the server) is the ideal way to overcome this issue.

**i.** From the EC2 cockpit, we click on **"Elastic Ips"**.

![elastic IPs](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2bd8c559-b18f-4db9-9446-84eed4c34c81)

**ii.** In the next page displayed, we click on **"Allocate Elastic IP Address"**.

![Allocate elastic IP address](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/633bb57d-7414-425e-ba73-66faa341eeb1)

**iii.** In the elastic IP address settings page we ensure to choose the same network border group as our EC2 instance. Then we select the option to choose from Amazon's pool of IPv4 addresses. Then we click on **"Allocate"**.

![elastic IP address settings](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2b22c080-2bfa-4b8f-aa59-808fc7d1ba9f)

**iv.** In the next page which shows us that the elastic IP has been allocated successfully, we click on the **"Actions'** drop down tab and we select **"Associate Elastic IP address"**.

![associate elatic ip 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/00c69fbe-bea1-435c-ba5a-37323e19ccc7)

**v.** In the Associate Elastic IP Address page, we scroll down to **"Instance"** and we select our EC2 instance. After this, we click on the **"Associate"** button at the bottom of the page.

![Associate Elastic IP address](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a8ee57a4-991e-4505-9fb8-9b7132353d8a)

**vi.** As can be seen in the output image below, the elastic IP address was successfully asssociated with our EC2 instance.

![Elastic IP successfully associated](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/a6dba97b-2953-485e-a13f-ecbe8ac62dd4)

#### <br>Step 4: Connect to the Jenkins-Ansible Server via the Terminal using the SSH Client<br/>

After we have provisioned our server and we have opened the necessary port, we must next connect to the server via an SSH client. This will enable us to subsequently be able to run commands on the terminal of our server. We carry this out by doing the following:

**i.** Download and Install an SSH client: Download and install [Termius](https://www.termius.com/download/windows) or Download and install [git](https://git-scm.com/downloads) (the ssh client - git bash will be packaged with the git installation)

**ii.** Establish connection with the EC2 instance: We connect to our EC2 instance via our Termius SSH client by following [these instructions:](https://dev.to/aws-builders/how-to-connect-your-ec2-linux-instance-with-termius-5209)

#### <br>Step 5: Jenkins Installation and Set-up<br/>

**i.** After connecting to our server we must first update all installed packages and their dependencies before commencing other installations or configurations. We do this by executing the following command: 

**`$ sudo apt update -y`**

![sudo apt update -y](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/5c7ab738-cee6-4385-b1f1-9ddff8f263af)

**ii.** Next, we run the following set of commands to [install dependencies for Jenkins](https://pkg.jenkins.io/debian-stable/):

```
$ sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \ https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

$ echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \ https://pkg.jenkins.io/debian-stable binary/ | sudo tee \ /etc/apt/sources.list.d/jenkins.list > /dev/null

$ sudo apt-get update

$ sudo apt-get install fontconfig openjdk-11-jre

```

![jenkins installation 1](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4e9df2c4-1a6c-41a8-9d51-18289cae592d)

**iii.** Then we execute the command below to install Jenkins:

**`$ sudo apt-get install jenkins`**

![jenkins installation 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4a53c019-5a7c-4d84-8999-4cb0ec297313)

**iv.** To ensure Jenkins is up and running, we enter the command below:

**`$ sudo systemctl status jenkins`**

![sudo systemctl jenkins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/d401bbae-7a7b-4e94-bbe6-08f8302dc06c)

**v.** Then we begin setting up Jenkins by accessing it via our browser using the following syntax: 

**`http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080`**

![unlock Jenkins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/730aadd0-bfb3-4774-b009-c7c7de539a2d)

**vi.** As shown in the output image above, we are prompted to provide an Administrator password to unlock Jenkins. To retrieve the password from the Jenkins Server, we enter the following command: 

**`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`**

![initial admin password](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b4b11c70-6079-4fef-a5bd-996b8959fc5a)

**vii.** Next, from the server, we copy the password as seen in the image above and we paste it in the dialogue box in the **"Unlock Jenkins"** page after which we click on **"Continue"**.

![unlock jenkins 2](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/ce0044ef-005e-498d-adc1-620b9a1f1e92)

**viii.** This brings us to the **"Customize Jenkins"** page. Here we select and click on **"Install Suggested Plugins"**.

![jenkins install suggested plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7b78280e-b378-4529-86db-be2804b09de0)

![installing plugins](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/4dd246cf-eb66-42e0-ac53-f038475ca8ca)

**ix.** As shown in the above image, we were able to successfully install the plugins. After completion, Jenkins prompts us to **"Create a First Admin User"** we can either fill in our details to do this or we just choose to **"Skip and continue as admin"**. We decide to go with the latter option.

![skip and continue as admin](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/82c59719-18a2-49b5-b1aa-610527076f30)

**x.** In the instance configuration page, we paste in our elastic ip address and click on **"Save and Finish"**.

![instance configuration](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/7a0e7627-7661-4af4-b270-d9efbca0207f)

**xi.** At this point the set-up is complete and Jenkins is ready to be used. We click on **"Start using Jenkins"** to move into the main Jenkins Environment.

![installation complete](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b725826c-ef8b-4906-b6ac-02d37cca0d43)

### <br>Install and Configure Ansible to act as a Jump Server/Bastion Host<br/>

The next phase of our project involves the installation and configuration of Ansible as a Jump Server/Bastion Host. An SSH Jump Server/Bastion Host is a regular Linux server, accessible from the Internet, which is used as a gateway to access other Linux machines on a private network using the SSH protocol. The purpose of an SSH jump server is to be the only gateway for access to our infrastructure reducing the size of any potential attack surface. We shall proceed to implement this with the following steps:

#### <br>Step 1: Create New Repository in GitHub<br/>

To create a new repository, we carry out the following steps:

**i.** We click on the plus sign **(+)** at the top right corner of our github account. A dropdown menu box appears and we select **"New repository"**.

![plus sign](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/b8fdf5ae-6f7a-4396-9bbc-aadaea9bfd2d)

**ii.** We fill out the form by entering **`ansible-config-mgt`** as name for our repository. We enter a description and then we check the box to add a README.md file. Afterwards, we leave every other box or button in their default state.

![create new repository](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/0f08b401-d40f-40a9-ae32-93757fa8c10d)

**iii.** We click the green **"Create repository"** button at the bottom of the page to create our repository.

![create repository button](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/2672317f-6cac-4411-ac17-319f512e2e10)

**iv.** As can be seen in the image below, we were able to successfuly create the repository.

![succesful create repository](https://github.com/QBDev0ps/DevOps-Cloud-projects/assets/140855364/8f358b9f-1417-4c5d-a5f3-d629a91651e6)

#### <br>Step 2: Ansible Installation<br/>

In this step, we are going to install Ansible on the same server (Jenkins-Ansible Server) where we have Jenkins installed.

**i.** First of all, we update the server machine if we have not already done so:

**`$ sudo apt update -y`**

**ii.** Then we install Ansible by executing the command below:

**`$ sudo apt install ansible`**

**iii.** Next we check our ansible version by running the following command:

**`$ ansible --version`**

**iv.** 
