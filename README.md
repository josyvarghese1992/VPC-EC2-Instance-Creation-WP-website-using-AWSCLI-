# VPC & EC2-Instance Creation using AWSCLI
<br />

Hopefully, this will be helpful for beginners who are interested in AWS.

Every organization faces a significant challenge in infrastructure management, from establishing the appropriate specifications to picking the appropriate tools to build and maintain it. Nowadays Terraform like tools are used by most businesses to manage their server infrastructure. Here I am starting with a  simple VPC & EC2 instance setup using the AWS CLI. 

Create a bastion, frontend(web server), and backend (db server) instance for this project. The backend server will be located in the private subnet and will connect to the internet through the NAT gateway. We will retain our bastion and front-end server in public subnets.

By keeping the database server in the private subnet and the web server in the public subnet, which can both be accessible from the bastion server, this configuration can be used to host a WordPress website.

### Important Notes:

- An IAM user with programmatic access, as well as complete VPC and EC2 permissions, is required.

- AWS CLI will be used to generate all resources.

##### *Firstly, An IAM user with EC2FullAccess, VPCFullAccess, and a machine with AWSCLI installed is required. Here I am providing the steps to create IAM user*

- Sign in as a root user. Provide username and password when prompted.

- Select the Users menu. Navigate to the Users screen. You'll find it in the IAM dashboard, under the Identity and Access Management (IAM) drop-down menu on the left side of the screen. Click on Users.

- Add a user. Click on Add User to navigate to a user detail form. Provide all details, such as the username and access type ( Select programmatic access). Click on Next: Permissions to continue.

- Set the user permissions. Click Attach existing policies directly,  and then filter the policies by keyword: AmazonEC2FullAccess, AmazonVPCFullAccess. Then click on the next add tag for the user, after that, you can see the user name with Access key ID and Secret access key save 

##### *Here, I have set up AWSCLI on an EC2 instance. So in essence, the EC2 instance will already have awscli installed. I'm going to use the commands listed below to update the AWSCLI*

````
$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
````
You can update the awscli using the below command :
````
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
````

***To configure AWScli, execute the command below***

````
$ aws configure
AWS Access Key ID [None]:XXXXXXXXXXXX 
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXX
Default region name [None]: us-east-2
Default output format [None]: json
````

#### ***Created a new VPC Assigned a tag with Name "awscli-webserver-vpc" using following commands***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-vpc --cidr-block 172.17.0.0/16 --query Vpc.VpcId --output text > AWSCLIVPCid.txt
[root@ip-172-31-41-74 ~]# cat AWSCLIVPCid.txt
vpc-0ae36a461ea8b749f
````

***Tags***
````
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources vpc-0ae36a461ea8b749f --tags Key=Name,Value=awscli-webserver-vpc
````


Make three subntes. One of them is a private subnet, and the other two are public subnets. In this case, I have make three subnets in the us-east-2a, us-east-2b, and us-east-2c regions, respectively.

<br />


- ***First subnet with a tag Name "awscli-webserver-public-1"***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-subnet --vpc-id vpc-0ae36a461ea8b749f --cidr-block 172.17.0.0/18 --availability-zone us-east-2a
````

*Output*

````
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "use2-az1",
        "AvailableIpAddressCount": 16379,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:us-east-2:457206769238:subnet/subnet-06c0ed5686b9673ab",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0ae36a461ea8b749f",
        "State": "available",
        "AvailabilityZone": "us-east-2a",
        "SubnetId": "subnet-06c0ed5686b9673ab",
        "OwnerId": "457206769238",
        "CidrBlock": "172.17.0.0/18",
        "AssignIpv6AddressOnCreation": false
    }
}
````
***Tags***
````
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources subnet-06c0ed5686b9673ab --tags Key=Name,Value=awscli-webserver-public-1
````

- ***Second subnet with a tag Name "awscli-webserver-public-2"***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-subnet --vpc-id vpc-0ae36a461ea8b749f --cidr-block 172.17.64.0/18 --availability-zone us-east-2b
````

*Output*

````
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "use2-az2",
        "AvailableIpAddressCount": 16379,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:us-east-2:457206769238:subnet/subnet-0b2c845c573620db7",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0ae36a461ea8b749f",
        "State": "available",
        "AvailabilityZone": "us-east-2b",
        "SubnetId": "subnet-0b2c845c573620db7",
        "OwnerId": "457206769238",
        "CidrBlock": "172.17.64.0/18",
        "AssignIpv6AddressOnCreation": false
    }
}
````

***Tags***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources subnet-0b2c845c573620db7 --tags Key=Name,Value=awscli-webserver-public-2
````

- ***Third subnet with a tag Name "awscli-webserver-private-1"***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-subnet --vpc-id vpc-0ae36a461ea8b749f --cidr-block 172.17.128.0/18 --availability-zone us-east-2c
````

*Output*

````
{
    "Subnet": {
        "MapPublicIpOnLaunch": false,
        "AvailabilityZoneId": "use2-az3",
        "AvailableIpAddressCount": 16379,
        "DefaultForAz": false,
        "SubnetArn": "arn:aws:ec2:us-east-2:457206769238:subnet/subnet-0b63487662232468d",
        "Ipv6CidrBlockAssociationSet": [],
        "VpcId": "vpc-0ae36a461ea8b749f",
        "State": "available",
        "AvailabilityZone": "us-east-2c",
        "SubnetId": "subnet-0b63487662232468d",
        "OwnerId": "457206769238",
        "CidrBlock": "172.17.128.0/18",
        "AssignIpv6AddressOnCreation": false
    }
}
````

***Tags***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources subnet-0b63487662232468d --tags Key=Name,Value=awscli-webserver-private-1
````

***Enabled to get public IP for the time of launch for the public subnets***

````
[root@ip-172-31-41-74 ~]# aws ec2 modify-subnet-attribute --subnet-id subnet-06c0ed5686b9673ab --map-public-ip-on-launch
[root@ip-172-31-41-74 ~]# aws ec2 modify-subnet-attribute --subnet-id subnet-0b2c845c573620db7 --map-public-ip-on-launch
````

***Created a internet gateway and attached to the VPC and tag it as "awscli-igw"***
<br />

Utilizing the internet gateway to route internet traffic into and out of the VPC. building the internet gateway in this stage.

````
[root@ip-172-31-41-74 ~]# aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
igw-0e37065885a906cb8

[root@ip-172-31-41-74 ~]# aws ec2 attach-internet-gateway --vpc-id vpc-0ae36a461ea8b749f --internet-gateway-id igw-0e37065885a906cb8
[root@ip-172-31-41-74 ~]#
````

***Tags***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources igw-0e37065885a906cb8 --tags Key=Name,Value=awscli-igw
````
***Created route tables***
<br />
To know where network traffic from our subnet or gateway is going, we need the route tables.

````
[root@ip-172-31-41-74 ~]# aws ec2 create-route-table --vpc-id vpc-0ae36a461ea8b749f --query RouteTable.RouteTableId --output text
rtb-04d66441006216548
````
***Tags***
````
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources  rtb-04d66441006216548 --tags Key=Name,Value=awscli-route-private
````

***Following command will help to provide the route table details***

````
[root@ip-172-31-41-74 ~]# aws ec2 describe-route-tables
````

***Renamed the default route table to the "awscli-route-default-public"***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources rtb-04a92da266a523f0c --tags Key=Name,Value=awscli-route-default-public
````

***Added that route table "awscli-route-default-public" to the default internetgateyway associated to the VPC***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-route --route-table-id rtb-04a92da266a523f0c --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0e37065885a906cb8
````

*Output*

````
{
    "Return": true
}
````

***Describe awscli-route-default-public route table***

````
[root@ip-172-31-41-74 ~]# aws ec2 describe-route-tables --route-table-id rtb-04a92da266a523f0c
````

*output*

````
{
    "RouteTables": [
        {
            "Associations": [
                {
                    "AssociationState": {
                        "State": "associated"
                    },
                    "RouteTableAssociationId": "rtbassoc-0f9e95055ba9e8210",
                    "Main": true,
                    "RouteTableId": "rtb-04a92da266a523f0c"
                }
            ],
            "RouteTableId": "rtb-04a92da266a523f0c",
            "VpcId": "vpc-0ae36a461ea8b749f",
            "PropagatingVgws": [],
            "Tags": [
                {
                    "Value": "awscli-route-default-public",
                    "Key": "Name"
                }
            ],
            "Routes": [
                {
                    "GatewayId": "local",
                    "DestinationCidrBlock": "172.17.0.0/16",
                    "State": "active",
                    "Origin": "CreateRouteTable"
                },
                {
                    "GatewayId": "igw-0e37065885a906cb8",
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "State": "active",
                    "Origin": "CreateRoute"
                }
            ],
            "OwnerId": "457206769238"
        }
    ]
}
````

#### ***Associated following First Public subnet ID to the default-public-route table***


````
[root@ip-172-31-41-74 ~]# aws ec2 associate-route-table  --subnet-id subnet-06c0ed5686b9673ab --route-table-id rtb-04a92da266a523f0c
````

*Output*

````
{
    "AssociationState": {
        "State": "associated"
    },
    "AssociationId": "rtbassoc-0f2e436fee12a40f3"
}
````

#### ***Associated following Second Public subnet ID to the default-public-route table***

````
[root@ip-172-31-41-74 ~]# aws ec2 associate-route-table  --subnet-id subnet-0b2c845c573620db7 --route-table-id rtb-04a92da266a523f0c
````

Output

````
{
    "AssociationState": {
        "State": "associated"
    },
    "AssociationId": "rtbassoc-07fb5b6f103a6ec93"
}
````


#### ***Allocated elastc IP***

An Elastic IP Address (EIP) is a public IP address that you can purchase and hold as an independent resource
<br />

````
[root@ip-172-31-41-74 ~]# aws ec2 allocate-address
````

*Output*

````
{
    "Domain": "vpc",
    "PublicIpv4Pool": "amazon",
    "PublicIp": "18.117.198.159",
    "AllocationId": "eipalloc-015b3dab03d415e86",
    "NetworkBorderGroup": "us-east-2"
}
````

#### ***configured nat-gateway with elastic IP allocation ID***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-nat-gateway \
>     --subnet-id subnet-0b2c845c573620db7 \
>     --allocation-id eipalloc-015b3dab03d415e86
````

*Output*

````
{
    "NatGateway": {
        "NatGatewayAddresses": [
            {
                "AllocationId": "eipalloc-015b3dab03d415e86"
            }
        ],
        "VpcId": "vpc-0ae36a461ea8b749f",
        "State": "pending",
        "NatGatewayId": "nat-04073779da21a76fb",
        "SubnetId": "subnet-0b2c845c573620db7",
        "CreateTime": "2022-12-09T09:06:56.000Z"
    },
    "ClientToken": "85d9b8bc-54e1-4827-a517-0fce7e8ed30f"
}
````

***Tags***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources nat-04073779da21a76fb --tags Key=Name,Value=awscli-nat
[root@ip-172-31-41-74 ~]#
````

#### ***Associated private subnet to the private route-table***

````
[root@ip-172-31-41-74 ~]# aws ec2 associate-route-table  --subnet-id subnet-0b63487662232468d --route-table-id rtb-04d66441006216548
{
    "AssociationState": {
        "State": "associated"
    },
    "AssociationId": "rtbassoc-0a5af0ce5b7489c20"
}

````

#### ***Created the rute rule for the private route-table with nat-gateway***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-route --route-table-id rtb-04d66441006216548 --destination-cidr-block 0.0.0.0/0 --nat-gateway-id nat-04073779da21a76fb
````

*Output*

````
{
    "Return": true
}
````

***To view the Descripition of the private route-table***

````
[root@ip-172-31-41-74 ~]# aws ec2 describe-route-tables --route-table-id rtb-04d66441006216548

````

*Output*

````
{
    "RouteTables": [
        {
            "Associations": [
                {
                    "SubnetId": "subnet-0b63487662232468d",
                    "AssociationState": {
                        "State": "associated"
                    },
                    "RouteTableAssociationId": "rtbassoc-0a5af0ce5b7489c20",
                    "Main": false,
                    "RouteTableId": "rtb-04d66441006216548"
                }
            ],
            "RouteTableId": "rtb-04d66441006216548",
            "VpcId": "vpc-0ae36a461ea8b749f",
            "PropagatingVgws": [],
            "Tags": [
                {
                    "Value": "awscli-route-private",
                    "Key": "Name"
                }
            ],
            "Routes": [
                {
                    "GatewayId": "local",
                    "DestinationCidrBlock": "172.17.0.0/16",
                    "State": "active",
                    "Origin": "CreateRouteTable"
                },
                {
                    "Origin": "CreateRoute",
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "NatGatewayId": "nat-04073779da21a76fb",
                    "State": "active"
                }
            ],
            "OwnerId": "457206769238"
        }
    ]
}
````


#### ***Description of subnets***

````
[root@ip-172-31-41-74 ~]# aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-0ae36a461ea8b749f" --query "Subnets[*].{ID:SubnetId,CIDR:CidrBlock}"
[
    {
        "CIDR": "172.17.0.0/18",
        "ID": "subnet-06c0ed5686b9673ab"
    },
    {
        "CIDR": "172.17.64.0/18",
        "ID": "subnet-0b2c845c573620db7"
    },
    {
        "CIDR": "172.17.128.0/18",
        "ID": "subnet-0b63487662232468d"
    }
]

````

#### ***Created security groups and inbound rules added***

We must now build the security group, which serves as a firewall. In this configuration bastion server and Web server will go in the public subnets and a database server that will go in the private subnet. Only port 22 for the bastion server is open, whereas port 80 for the front-end server will accept traffic from everywhere, and port 22 for the Web server and DB server from the bastion server only. Also, DB server is reachable from the bastion server and port 3306 from the Web server.

For these circumstances, Here I have generated 3 security groups to comply with the criteria mentioned earlier.

*Security group for bastion server*

````
[root@ip-172-31-41-74 ~]# aws ec2 create-security-group --group-name sshaccess --description "Security group for SSH access" --vpc-id vpc-0ae36a461ea8b749f
````

*Output

````
{
    "GroupId": "sg-0d3ff61774000ef97"
}
````

*Security group for the web server*


````
[root@ip-172-31-41-74 ~]# aws ec2 create-security-group --group-name webserveraccess --description "Security group for Webserver access" --vpc-id vpc-0ae36a461ea8b749f
````
*Output*

````
{
    "GroupId": "sg-0a0fc9eacd4a82cbf"
}
````

*Security group for the database server*

````
[root@ip-172-31-41-74 ~]# aws ec2 create-security-group --group-name dbaccess --description "Security for DB access" --vpc-id vpc-0ae36a461ea8b749f
````

*Output*

````
{
    "GroupId": "sg-0009850e3e9dd8339"
}
````

***Tags***

````
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources sg-0d3ff61774000ef97 --tags Key=Name,Value=awscli-ssh-sec
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources sg-0a0fc9eacd4a82cbf --tags Key=Name,Value=awscli-web-sec
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources sg-0009850e3e9dd8339 --tags Key=Name,Value=awscli-db-sec
````

*A rule to allow SSH access to the bastion server*

````
[root@ip-172-31-41-74 ~]# aws ec2 authorize-security-group-ingress --group-id sg-0d3ff61774000ef97 --protocol tcp --port 22 --cidr 0.0.0.0/0
````

*A rule to allow HTTP access to the web server from the public network*

````
[root@ip-172-31-41-74 ~]# aws ec2 authorize-security-group-ingress --group-id sg-0a0fc9eacd4a82cbf --protocol tcp --port 80 --cidr 0.0.0.0/0
````

*A rule to allow HTTPS access to the web server from the public network*

````
[root@ip-172-31-41-74 ~]# aws ec2 authorize-security-group-ingress --group-id sg-0a0fc9eacd4a82cbf --protocol tcp --port 443 --cidr 0.0.0.0/0
````

*A rule to allow SSH access to the web server from the bastion server*

````
[root@ip-172-31-41-74 ~]# aws ec2 authorize-security-group-ingress --group-id sg-0a0fc9eacd4a82cbf --protocol tcp --port 22 --source-group sg-0d3ff61774000ef97
````

*A rule to allow SSH access to the db server from the bastion server*

````
[root@ip-172-31-41-74 ~]# aws ec2 authorize-security-group-ingress --group-id sg-0009850e3e9dd8339 --protocol tcp --port 22 --source-group sg-0d3ff61774000ef97
````

*A rule to enable the web server to access mysql on the db server*

````
[root@ip-172-31-41-74 ~]# aws ec2 authorize-security-group-ingress --group-id sg-0009850e3e9dd8339 --protocol tcp --port 3306 --source-group sg-0a0fc9eacd4a82cbf

````

#### ***Created a key***
Using the command below, create a key pair and save it to a file. Additionally, changed the file's permissions.

````
[root@ip-172-31-41-74 ~]# aws ec2 create-key-pair --key-name awsclinewkey.pem --query "KeyMaterial" --output text >  awsclinewkey.pem
[root@ip-172-31-41-74 ~]# chmod 400  awsclinewkey.pem
````


#### ***Created userdata files***

*Created web.sh file and inserted below code*

````
#!/bin/bash


echo "ClientAliveInterval 60" >> /etc/ssh/sshd_config
echo "LANG=en_US.utf-8" >> /etc/environment
echo "LC_ALL=en_US.utf-8" >> /etc/environment
service sshd restart

yum install httpd -y
amazon-linux-extras install php7.4
systemctl restart httpd.service
systemctl enable httpd.service

````

*Created bastion.sh file and inserted below code*

````
#!/bin/bash


echo "ClientAliveInterval 60" >> /etc/ssh/sshd_config
echo "LANG=en_US.utf-8" >> /etc/environment
echo "LC_ALL=en_US.utf-8" >> /etc/environment
service sshd restart
````

*Created db.sh file and inserted below code*

````
#!/bin/bash


echo "ClientAliveInterval 60" >> /etc/ssh/sshd_config
echo "LANG=en_US.utf-8" >> /etc/environment
echo "LC_ALL=en_US.utf-8" >> /etc/environment
service sshd restart
yum install mariadb-server -y
````

#### ***Launch instance webserver***

````
[root@ip-172-31-41-74 ~]# aws ec2 run-instances --image-id ami-0beaa649c482330f7 --count 1 --instance-type t2.micro --key-name awsclinewkey.pem --security-group-ids sg-0a0fc9eacd4a82cbf --subnet-id subnet-06c0ed5686b9673ab --user-data file://web.sh
````


#### ***Launch instance bastion*** 

````
[root@ip-172-31-41-74 ~]# aws ec2 run-instances --image-id ami-0beaa649c482330f7 --count 1 --instance-type t2.micro --key-name awsclinewkey.pem --security-group-ids sg-0d3ff61774000ef97 --subnet-id subnet-0b2c845c573620db7 --user-data file://bastion.sh
````


#### ***Launch instance datbase server***

````
[root@ip-172-31-41-74 ~]# aws ec2 run-instances --image-id ami-0beaa649c482330f7 --count 1 --instance-type t2.micro --key-name awsclinewkey.pem --security-group-ids sg-0009850e3e9dd8339  --subnet-id subnet-0b63487662232468d --user-data file://db.sh
````

**We can get the instance ID of each instances from the output of above commands.

*Tags*

````
[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources i-09faef9c80b28b17d --tags Key=Name,Value=awscli-dbserver

[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources i-0876644afd80003a0 --tags Key=Name,Value=awscli-bastion

[root@ip-172-31-41-74 ~]# aws ec2 create-tags \
>     --resources i-0d2e33679e51880fe --tags Key=Name,Value=awscli-webserver
````

#### ***Instance description***
The following command will list the public IP of each instance.

````
[root@ip-172-31-41-74 ~]# aws ec2 describe-instances --instance-id i-09faef9c80b28b17d --query "Reservations[*].Instances[*].{State:State.Name,Address:PublicIpAddress}"

[root@ip-172-31-41-74 ~]# aws ec2 describe-instances --instance-id i-0876644afd80003a0 --query "Reservations[*].Instances[*].{State:State.Name,Address:PublicIpAddress}"

[root@ip-172-31-41-74 ~]# aws ec2 describe-instances --instance-id i-0d2e33679e51880fe --query "Reservations[*].Instances[*].{State:State.Name,Address:PublicIpAddress}"

````

#### ***enabled dnshostname for VPC***
This command will provide the public DNS hostname for the instances which have public IP

````
[root@ip-172-31-41-74 ~]# aws ec2 modify-vpc-attribute --vpc-id vpc-0ae36a461ea8b749f --enable-dns-hostnames "{\"Value\":true}"
[root@ip-172-31-41-74 ~]# aws ec2 describe-vpc-attribute --vpc-id vpc-0ae36a461ea8b749f --attribute enableDnsHostnames
````

*Output*

````
{
    "VpcId": "vpc-0ae36a461ea8b749f",
    "EnableDnsHostnames": {
        "Value": true
    }
}

````


***Connected to bastion***

````
[root@ip-172-31-41-74 ~]# ssh -i awsclinewkey.pem ec2-user@3.12.102.219
[ec2-user@ip-172-17-76-191 ~]$ sudo su -


[root@ip-172-17-76-191 ~]# vi awsclinewkey.pem
[root@ip-172-17-76-191 ~]# chmod 400 awsclinewkey.pem

````

***ssh from bastion to database server***

````
[root@ip-172-17-76-191 ~]# ssh -i awsclinewkey.pem ec2-user@172.17.138.85

[root@ip-172-17-138-85 ~]# systemctl restart mariadb.service
[root@ip-172-17-138-85 ~]# systemctl enable mariadb.service
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.

[root@ip-172-17-138-85 ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password:
Re-enter new password:
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

[root@ip-172-17-138-85 ~]# mysql -u root -pmysqlroot123
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 10
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database blog;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> create user 'bloguser'@'%' identified by 'bloguser123';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> grant all privileges on blog.* to 'bloguser'@'%';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> exit
Bye
[root@ip-172-17-138-85 ~]#
````

*Created db, dbuser, dbuser password*


***ssh from bastion to webserver***

````
[root@ip-172-17-76-191 ~]# ssh -i awsclinewkey.pem ec2-user@172.17.5.195

[ec2-user@ip-172-17-5-195 ~]# sudo su -

[root@ip-172-17-5-195 ~]# wget https://wordpress.org/wordpress-6.1.zip
--2022-12-09 18:56:00--  https://wordpress.org/wordpress-6.1.zip
Resolving wordpress.org (wordpress.org)... 198.143.164.252
Connecting to wordpress.org (wordpress.org)|198.143.164.252|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 24365389 (23M) [application/zip]
Saving to: ‘wordpress-6.1.zip’

100%[========================================================================================================================>] 24,365,389  11.2MB/s   in 2.1s

2022-12-09 18:56:02 (11.2 MB/s) - ‘wordpress-6.1.zip’ saved [24365389/24365389]

[root@ip-172-17-5-195 ~]# unzip wordpress-6.1.zip
Archive:  wordpress-6.1.zip
   creating: wordpress/
[root@ip-172-17-5-195 ~]# cp -pr wordpress/* /var/www/html/
[root@ip-172-17-5-195 ~]# chown -R apache. /var/www/html/
[root@ip-172-17-5-195 ~]# mv /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
[root@ip-172-17-5-195 ~]# vi /var/www/html/wp-config.php
[root@ip-172-17-5-195 ~]# systemctl restart httpd.service
[root@ip-172-17-5-195 ~]# vi /var/www/html/wp-config.php
[root@ip-172-17-5-195 ~]#
````

*Hosted a wordpress site, We can access the site using the public IP of the awscli-webserver*

*Informational note: Using Route53, or using with any other namesrvers, we can able to add the public IP address of the "awscli-webserver" as an A record for any of your domain. Then make the necessary updates to "wp home" and "siteurl" in the Wordpress configuration file for the subdomain entry*
