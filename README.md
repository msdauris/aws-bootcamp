# aws-essentials

A workshop

#AWS Account
https://730492949633.signin.aws.amazon.com/console

# AWS Region
eu-west-1 Ireland

| **Team**       | **Users**      | **Users**     | **Users** | **Users** |
|----------------|----------------|---------------|-----------|-----------|
| Admins         | soyred_admin   |               |           |           |
| Team 1         | Team1-User1    | Team1-User2   |  etc      |    etc    |
| Team 2         | Team2-User1    | Team2-User2   |  etc      |    etc    |

### CIDR Blocks, Availability Zones, Subnets and Route Tables

| **Team**       | **CIDR Block**  | **Availability Zone** | **Public Subnet**     | **Private Subnet**    | **Public Route Table**                | **Private Route Table**                            |
|----------------|-----------------|-----------------------|-----------------------|-----------------------|---------------------------------------|----------------------------------------------------|
| Demo Account   | 10.0.0.0/16     | eu-west-1c            | 10.0.0.0/24           | 10.0.16.0/20          | 0.0.0.0/0 via Internet Gateway        | 0.0.0.0/0 via NAT Instance/Gateway or VPC Endpoint |
| Team 1         | 10.2.0.0/16     | eu-west-1a            | 10.2.0.0/24           | 10.2.16.0/20          | 0.0.0.0/0 via Internet Gateway        | 0.0.0.0/0 via NAT Instance/Gateway or VPC Endpoint |
| Team 2         | 10.3.0.0/16     | eu-west-1b            | 10.3.0.0/24           | 10.3.16.0/20          | 0.0.0.0/0 via Internet Gateway        | 0.0.0.0/0 via NAT Instance/Gateway or VPC Endpoint |
| Team 3         | 10.4.0.0/16     | eu-west-1c            | 10.4.0.0/24           | 10.4.16.0/20          | 0.0.0.0/0 via Internet Gateway        | 0.0.0.0/0 via NAT Instance/Gateway or VPC Endpoint |
| Additional     | 10.5.0.0/16     |                       | 10.5.0.0/24           | 10.5.16.0/20          | 0.0.0.0/0 via Internet Gateway        | 0.0.0.0/0 via NAT Instance/Gateway or VPC Endpoint |


### Security Group Rules

| **Team**       | **Security Group**    | **Inbound Rules**                                                                                                                      | **Outbound Rules**            |
|----------------|-----------------------|----------------------------------------------------------------------------------------------------------------------------------------|-------------------------------|
| All Accounts   | Public SG             | Allow SSH (22) from your IP (Restricts EC2 Connect) & Allow HTTP 80 from  (0.0.0.0/0)                                                  | Allow all traffic (0.0.0.0/0) |
|                | Private SG            | Allow SSH (22) from Public SG/Private IP of Public aka Bastion Host                                                                    | Allow all traffic (0.0.0.0/0) |
|                | Nat SG                | Allow HTTP (80) from VPC CIDR<br>Allow HTTPS (443) from VPC CIDR<br>Allow SSH (22) from VPC CIDR<br>Allow ALL ICMP-IPv4 from 0.0.0.0/0 | Allow all traffic (0.0.0.0/0) |

