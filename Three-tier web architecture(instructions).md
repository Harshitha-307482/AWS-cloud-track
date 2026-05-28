**Three-tier web architecture**



**1) Create VPC**

&#x20;IPv4 CIDR - 10.0.0.0/16



**2) Subnets**

name: 3-tier-PubSub-A

Availability Zone: us-east-1a

IPv4 subnet CIDR block: 10.0.1.0/24



name: 3-tier-PubSub-B

Availability Zone: us-east-1b

IPv4 subnet CIDR block: 10.0.2.0/24



name: 3-tier-PrivateSub-A

Availability Zone: us-east-1a

IPv4 subnet CIDR block: 10.0.11.0/24



name: 3-tier-PrivateSub-B

Availability Zone: us-east-1b

IPv4 subnet CIDR block: 10.0.12.0/24



name: 3-tier-DBPvtSub-A

Availability Zone: us-east-1a

IPv4 subnet CIDR block: 10.0.21.0./24



name: 3-tier-DBPvtSub-B

Availability Zone: us-east-1b

IPv4 subnet CIDR block: 10.0.22.0/24



**3)Internet gateway**

&#x20;name: 3-tier-IGW



Attach to VPC that we created



**4) NAT**

&#x20;name: 3-Tier-NAT

&#x20;Availability mode: Zonal

&#x20;Subnet: 3-tier-pubsub-A

&#x20;Connectivity type: public

&#x20;Allocate Elastic IP



**5) Create route table**

&#x20;name: 3-Tier-Pub-RT

&#x20;select VPC

&#x20;-edit-routes:

&#x20; destinations: 0.0.0.0/0

&#x20; target: internet gateway

&#x20; select 3-tier-IGW



&#x20;name: 3-Tier-PVT-RT

&#x20;-edit-routes:

&#x20;destinations: 0.0.0.0/0

&#x20; target: NAT gateway

&#x20; select 3-tier-NAT



&#x20;name: 3-tier-DBPVT-RT

&#x20;-edit-routes:

&#x20;destinations: 0.0.0.0/0

&#x20; target: NAT gateway

&#x20; select 3-tier-NAT

&#x20;

**6) Associate Subnets**

&#x20;route tables >

&#x20;

&#x20;associate-pub-rt

&#x20;pubA

&#x20;pubB

&#x20;

&#x20;associate-pvt-rt

&#x20;pvtA

&#x20;pvtB



&#x20;associate-DB-rt

&#x20;db-rt

&#x20;db-rt



**7) Security groups**

&#x20; Security group name: 3-tier-ALB

&#x20; VPC: VPC-three-tier

&#x20; Inbound rules:

&#x20;  type: HTTP

&#x20;  anywhere-ipv4

&#x20;  source 0.0.0.0/0



&#x20; Security group name: 3-tier-Bastion

&#x20; VPC: VPC-three-tier

&#x20; Inbound rules:

&#x20;  type: SSH

&#x20;  anywhere-ipv4

&#x20;  source 0.0.0.0/0



&#x20; Security group name: 3-tier-Frontend

&#x20; VPC: VPC-three-tier

&#x20; Inbound rules:

&#x20;  a)type: HTTP

&#x20;    custom

&#x20;    source: 3-Tier-ALB

&#x20;  b)type: SSH

&#x20;    custom

&#x20;    source: 3-Tier-Bastion



&#x20; Security group name: 3-tier-internal-ALB

&#x20; VPC: VPC-three-tier

&#x20; Inbound rules:

&#x20;  type: HTTP

&#x20;  custom

&#x20;  source: 3-tier-frontend



&#x20; Security group name: 3-tier-Backend

&#x20; VPC: VPC-three-tier

&#x20; Inbound rules:

&#x20;  a)type: HTTP

&#x20;    custom

&#x20;    source: 3-Tier-internal-ALB

&#x20;  b)type: SSH

&#x20;    custom

&#x20;    source: 3-Tier-Bastion



&#x20; Security group name: 3-tier-DB

&#x20; VPC: VPC-three-tier

&#x20; Inbound rules:

&#x20;  a)type: MYSQL/Aurora

&#x20;    custom

&#x20;    source: 3-Tier-Backend

&#x20;  b)type: SSH

&#x20;    custom

&#x20;    source: 3-Tier-Bastion



**8) Create target group**

&#x20;  Target type: Instances

&#x20;  Target group name: 3-Tier-External-LB

&#x20;  Protocol: HTTP

&#x20;  Port: 80

&#x20;  VPC: VPC-three-tier



&#x20;  Target type: Instances

&#x20;  Target group name: 3-Tier-Internal-LB

&#x20;  Protocol: HTTP

&#x20;  Port: 80

&#x20;  VPC: VPC-three-tier



**9) Create Application Load balancer**

&#x20;  Load balancer name: 3-Tier-External-LB

&#x20;  Scheme: Internet-facing

&#x20;  VPC: VPC-three-tier

&#x20;  Availability Zones and subnets:

&#x20;   us-eat-1a  > 3-tier-PubSub-A

&#x20;   us-eat-1b  > 3-tier-PubSub-B

&#x20;  Security groups:

&#x20;    3-tier-ALB

&#x20;  Listeners and routing:

&#x20;    HTTP 80

&#x20;  Default action:

&#x20;     Forward to target groups

&#x20;     Target group: 3-Tier-External-LB



&#x20;  Load balancer name: 3-Tier-Internal-LB

&#x20;  Scheme: Internal

&#x20;  VPC: VPC-three-tier

&#x20;  Availability Zones and subnets:

&#x20;   us-eat-1a  > 3-tier-PvtSub-A

&#x20;   us-eat-1b  > 3-tier-PvtSub-B

&#x20;  Security groups:

&#x20;    3-tier-internal-ALB

&#x20;  Listeners and routing:

&#x20;    HTTP 80

&#x20;  Default action:

&#x20;     Forward to target groups

&#x20;     Target group: 3-Tier-internal-LB





**10) Launch an instance:**



**3-tier-Bastion**



| Setting               | Value                  |

| --------------------- | ---------------------- |

| Name                  | 3-tier-Bastion         |

| AMI                   | Amazon Linux 2023      |

| Instance Type         | t2.micro               |

| Key Pair              | Create/select existing |

| VPC                   | VPC-three-tier         |

| Subnet                | 3-tier-PubSub-A        |

| Auto Assign Public IP | ENABLE                 |

| Security Group        | 3-tier-Bastion         |

| IAM Role              |                        |

\--------------------------------------------------



**CREATE WEB EC2**



| Setting               | Value               |

| --------------------- | ------------------- |

| Name                  | web-tier-server     |

| AMI                   | Amazon Linux 2023   |

| Type                  | t2.micro            |

| VPC                   | VPC-three-tier      |

| Subnet                | 3-tier-PrivateSub-A |

| Auto Assign Public IP | ENABLE              |

| Security Group        | 3-tier-Frontend     |

| IAM Role              |                     |

&#x20;----------------------------------------------



**LAUNCH APP EC2**



| Setting   | Value               |

| --------- | ------------------- |

| Name      | app-tier-server     |

| AMI       | Amazon Linux 2023   |

| Type      | t2.micro            |

| VPC       | VPC-three-tier      |

| Subnet    | 3-tier-PrivateSub-A |

| Public IP | DISABLE             |

| SG        | 3-tier-Backend      |

| Key Pair  | Same existing key   |











CONNECT TO BASTION

ssh -i AWS.pem ec2-user@BASTION-PUBLIC-IP





CONNECT TO WEB TIER FROM BASTION

ssh -i AWS.pem ec2-user@10.0.1.118

OPEN WEB TIER FILES

Open nginx config

sudo nano /etc/nginx/nginx.conf

Restart nginx

sudo systemctl restart nginx

Check nginx status

sudo systemctl status nginx

Test Internal ALB

curl http://internal-3-Tier-Internal-LB-1722000809.us-east-1.elb.amazonaws.com/health

Test transaction API

curl http://internal-3-Tier-Internal-LB-1722000809.us-east-1.elb.amazonaws.com/transaction





CONNECT TO APP TIER FROM BASTION

ssh -i AWS.pem ec2-user@10.0.11.176

OPEN APP TIER FILES

Open backend app

nano \~/app-tier/index.js

Open DB config

nano \~/app-tier/DbConfig.js

Open Transaction Service

nano \~/app-tier/TransactionService.js

PM2 COMMANDS

Start app

pm2 start index.js

Restart app

pm2 restart index

Check app status

pm2 status

View logs

pm2 logs

TEST BACKEND

Health check

curl http://localhost:4000/health

Transaction API

curl http://localhost:4000/transaction





CONNECT TO DB TIER

ssh -i AWS.pem ec2-user@10.0.21.79

MYSQL COMMANDS

Login MySQL

mysql -u root -p

Restart MySQL

sudo systemctl restart mysqld

Check MySQL status

sudo systemctl status mysqld

Open MySQL config

sudo nano /etc/my.cnf



domain, route 53, 









&#x20;

