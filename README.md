# VPCs, subnets, securtiy groups and more.

### Create VPC on AWS

- Name VPC
- **IPv4 CIDR block** -> add ip (example 10.201.0.0/16)
- **Create VPC**
<br> </br>

- ----------------------------------
### Create Internet Gateway
- **Name tag** -> Name your internet gateway
- **Create internet gateway**
<br> </br>
- Once created, select your internet gateway 
- Select **Actions** -> **Attach to VPC**
- Attach to VPC  you just created
<br> </br>

- ----------------------------------------------
### Create subnets
- Create two subnets: 
	- Add your VPC to attach to both subnets



- **Public subnet**:
	- Name public subnet
	- **Availability zone** -> **Europe(ireland) / eu-west-1a**
	- **IPv4 CIDR block** -> Add IP (example 10.201.1.0/24)
<br></br>

- **Private subnet**:
	- Name private subnet
	- **Availability zone** -> **Europe(ireland) / eu-west-1a**
	- **IPv4 CIDR block** -> Add IP (example 10.201.2.0/24)

- **Create Subnet**
<br> </br>

- -----------------------------------
### Create Route Table
- Name route table
- **VPC** -> your VPC
- **Create route table**
<br> </br>
- **Select table and Edit routes**
- **Add route**:
	- **Destination** -> 0.0.0.0/0 
	- **Target** -> **Internet Gateway** -> select your internet gateway
- **Save changes**
<br> </br>

- -----------------------------
### Connectng route table to public subnet
- Select public subnet
- Select **route table** -> **Edit route table association**
- Under **Route table ID**:
	 - Change **Main route table** to your route table
- **Save**
<br> </br>

- -----------------------------------------------
### Create Securty Groups 

- Application security group:
<br> </br>

	- Name security group
	- Add description 
	- **VPC** -> your VPC
	- **Inbound rules**:
		- **Type** -> HTTP, **Source type** -> Anywhere-IPv4
		- **Type** -> SSH, **Source type** -> My IP
		- **Port range** -> 3000, **Source** -> 0.0.0.0/0 (Will be needed later on to allow for app reverse proxy)
- **Create security group**
<br> </br>
- Data Base security group:
<br> </br>

	- Name security group
	- Add description 
	- **VPC** -> your VPC
	- **Inbound rules**:
		- **Type** -> Custom TCP, **Port range** -> 27017, **Source type** -> enter the name of your app security group and select the option that comes up
		

- **Create security group**

<br> </br>
- -----------------------------------------
### Create nerwork ACLs
- **Create Public ACL**
	- Name your public network ACL
	- **VPC** -> your VPC
	- **Create network ACL**
<br> </br>
- Select and open your public ACL
<br> </br>
	- **Inbound rules** -> Edit Inbound rules:
		- **Rule number** -> 100, **Type** -> HTTP
		- **Rule number** -> 110, **Type** -> SSH, **Source**- > your.public.IP/32
		- **Rule number** -> 120, **Port range** -> 1024-65535 

- **Save changes**
<br> </br>

	- **Outbound rules** -> Edit outbound rules:
		- **Rule number** -> 100, **Type** -> HTTP
		- **Rule number** -> 110, **Port range** -> 27017, **Destination** -> private.subnet.ip/24   (this case 10.201.2.0/24)
		- **Rule number** -> 120, **Port range** -> 1024-65535
- **Save changes**

<br> </br>

- **Create Private ACL**
	- Name your private network ACL
	- **VPC** -> your VPC
	- **Create network ACL**
<br> </br>
- Select and open your private ACL
<br> </br>
	- **Inbound rules** -> Edit Inbound rules:
		- **Rule number** -> 100, **Type** -> HTTP
		- **Rule number** -> 110, **Port range** -> 27017, **Source** ->  public.subnet.ip/24   (this case 10.201.1.0/24) 

- **Save changes**
<br> </br>

	- **Outbound rules** -> Edit ourbound rules:
		- **Rule number** -> 100, **Type** -> HTTP
		- **Rule number** -> 110, **Port range** -> 1024-65535, **Destination** ->  public.subnet.ip/24   (this case 10.201.1.0/24)
- **Save changes**

<br> </br>
- **Once you have created both ACLs**
<br> </br>
- Open up private ACL:
	- **Subnet association**
	- **Edit Subnet associations**
	- Select public subnet

<br> </br>
- Open up public ACL:
	- **Subnet association**
	- **Edit Subnet associations**
	- Make sure only public subnet is selected
<br> </br>
- ---------------------------------------
### Create a webserver server EC2 instance

**IF YOU ALREADY HAVE AN AMI INSTANCE OF THE APP SERVER, LAUNCH IT AND FOLLOW BELOW STEPS WHERE NEDED**
<br> </br> 
- **Choose AMI**
	- Select **Ubuntu Server 16.04 LTS, 64-bit**
- **Choose an Instant Type**
	- Select **t2 micro**
- **Configure Instance**
	- **Network** -> your VPC
	- **Subnet** -> your public subnet
	- **Auto-assign public IP** -> enable
- **Configure Security Group**
	- Select **Select an EXISTING security group**
	- Select you app security group. It should allow port 80 and 22
- **Review and Launch** -> **Launch**
- **Select an existing key pair...** -> Select your own key (In my case eng89_devops.pem)
- **Launch Instance**

<br> </br>

- ------------------------------------------
### Copy app folder from local host to app server instance
- DO THIS STEP ONLY IF YOU'RE CREATEING APP INSTANCE FROM SCRATCH
<br> </br>
- In your terminal, navigate to the directory with the app folder
- Run command:
``` scp -i ~/.ssh/eng89_devops.pem -r app/ ec2-user@APP.PUBLIC.SERVER.IP:~/app/```

- Once copy complete, you should have app folder in your app instance
<br> </br>
- -----------------------------------------
### Create a database server EC2 instance 

**IF YOU ALREADY HAVE AN AMI INSTANCE OF THE DB SERVER, LAUNCH IT AND FOLLOW BELOW STEPS WHERE NEEDED**
<br> </br>
- **Choose AMI**
	- Select **Ubuntu Server 16.04 LTS, 64-bit**
- **Choose an Instant Type**
	- Select **t2 micro**
- **Configure Instance**
	- **Network** -> your VPC
	- **Subnet** -> your private subnet
	- **Auto-assign public IP** -> Use subnet setting (Disable)
- **Configure Security Group**
	- Select **Select an EXISTING security group**
	- Select you database security group. It should allow port 27017
- **Review and Launch** -> **Launch**
- **Select an existing key pair...** -> Select your own key (In my case eng89_devops.pem)
- You will not be able to ssh into this server just yet
<br> </br>

- -----------------------------------
### Create Bastion public subnet 
- **Create subnet**
	- **VPC** -> your vpc
	- **Subnet name** -> name your subnet
	- **Availability zone** -> ...eu-west-1a
	- **IPv4 CIDR block** -> Add IP (example 10.201.3.0/24)
	- **Create subnet**
<br> </br>

- Edit **route table**
- (Should be same as public subnet)
	- **Edit route table association**
	- **route table ID** -> the route table you created
	- **Save**
<br> </br>
- ---------------------------------
### Create new network ACL for Bastion subnet
- **Create network ACL**
	- **Name** -> name the ACL
	- **VPC** -> your VPC
- **Create nework ACL**
<br> </br>
- Open and select new ACL
- **Edit inbound rules**
	- **Rule number** -> 100, **Type** -> SSH, **Source** -> your.public.ip/32
	- **Rule number** -> 110, **Port range** -> 1024-65535 **Source** ->db.subnet.ip/24

- **Save changes**
<br> </br>
- **Edit outbound rules**
	- **Rule number** -> 100, **Type** -> SSH, **Destination** -> db.subnet.ip/24
	- **Rule number** -> 110, **Port range** -> 1024-65535, **Destination** -> your.public.ip/32
- **Save changes**
<br> </br>
- ----------------------------------------
### Create new Bastion security group 
- **Create new security group**
- Name security group
- **VPC** -> add your VPC
<br> </br>
- **Add inbound rules**
	- **Type** -> SSH, **Source** -> your.public.ip/32
	- **Create security group**
<br> </br>
- --------------------------------------
### Create a Bastion server EC2 instance 
- **Choose AMI**
	- Select **Ubuntu Server 16.04 LTS, 64-bit**
- **Choose an Instant Type**
	- Select **t2 micro**
- **Configure Instance**
	- **Network** -> your VPC
	- **Subnet** -> your Bastion subnet
	- **Auto-assign public IP** -> Enable
- **Configure Security Group**
	- Select **Select an EXISTING security group**
	- Select you Bastion security group. It should allow port 22
- **Review and Launch** -> **Launch**
- **Select an existing key pair...** -> Select your own key (In my case eng89_devops.pem)
<br> </br>
- **COPY YOUR .PEM KEY INTO THE BASTION SERVER**
- Make sure you're in directory with the .pem key
- Run below command to copy over to server
```
 scp -i ~/.ssh/eng89_devops.pem -r eng89_devops.pem/ ubuntu@PUBLIC.BASTION.SERVER.IP:~/

```
<br> </br>
- Once the key is in the Bastion server, move it to the .ssh file
- Run `ls -ls` to check if .ssh exists
- To move key, run: `mv name_of_key.pem .ssh/
`
- ---------------------------------
### Updating Private network ACL's inbound and outbound rules to allow Bastion access

- **Edit inbound rules**
	- **Rule number** -> 120, **Type** -> SSH, **Source** -> Bastion.public.ip/24 (In my case 10.201.3.0/24)
	<br> </br>
- **Edit Outbound rules**
	- **Rule number** -> 120, **Port range** -> 1024-65535, **Destination** -> Bastion.public.ip/24
- **Save changes**
<br> </br>
- --------------------------------------
### Updating Database security group inbound rules to allow Bastion access
- **Edit inbound rules**
 	- **Type** -> SSH, **Source** -> Bastion securty group
- **Save rules**
<br> </br>
- ----------------------------------
### Setting up instances and launching Sparta App

- Start your App, Database and Bastion instances
- SSH into App and Bastion instance
	- If you created your instances from AMIs, remember to change `root` -> `ubuntu` when running SSH command
- In Bastion instace, navigate to the file with your .pem key (In my case `cd .ssh/`)
	- SSH into Database from whithin Bastion instance
<br> </br>
- **Setting up App and DB**
	- If you used your app/db instace AMIs, you will not need to install mongod, nginx etc.
	- If you started your instances from scratch, follow these steps to get up App and DB: https://github.com/NikiNikiforidi/eng89_2tier_app_deployment
<br> </br>
<br> </br>
- **Set up reverse proxy in App instance, if from AMI**
	- Navigate to `cd /etc/nginx/sites-available`
	- Change default file to:
```
	server {
    listen 80;

    server_name _;
    location / {
        proxy_pass http://APP.SERVER.PUBLIC.IP:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
	


```


	- To chech nginx config, run: `sudo nginx -t`
	- Restart nginx `sudo systemctl restart nginx`
- To check it working enter, `app_public_ip` in browser
<br> </br>
- **Update persistant veriable**
	- To create, OR override old variable, run:
	`sudo echo export DB_HOST="mongodb://DB.SERVER.PRIVATE.IP:27017/posts" >> ~/.bashrc`
	<br> </br>
	- Then run: 
	`source ~/.bashrc`
	- To check if varaible exists, run `env` or `printenv DB_HOST`
<br> </br>
- **Run WebApp**
	- In App instance, navigate to `cd app/` and run `npm start` 
	- To check it working enter, `app_public_ip/posts` in browser