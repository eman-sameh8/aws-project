# Scalable Web Application with ALB and Auto Scaling
![image](https://github.com/user-attachments/assets/a5d374e9-eb75-480b-a77f-985fb5db518f)


In my design I used multiple Availability Zones within a single AWS Region to ensure redundancy, fault tolerance, and scalability.

At the core of this architecture is a Virtual Private Cloud (VPC) with a CIDR block of 10.0.0.0/16. This VPC provides a logically isolated network in which all resources are launched. The network is divided into multiple subnets: public subnets for web servers and private subnets for databases.

Two Availability Zones are used to enhance availability and fault tolerance. Each zone contains one public subnet and one private subnet. The public subnets (10.0.1.0/24 and another similar range in the second AZ) host an Auto Scaling Group of EC2 instances. These instances serve as web servers and are placed in public subnets so they can handle incoming HTTP/HTTPS requests from users through the app load balancer. The Auto Scaling Group ensures that the number of EC2 instances automatically adjusts based on traffic, providing both performance and cost-efficiency.

In front of the Auto Scaling Group, a Load Balancer is used to distribute traffic evenly across the EC2 instances. This improves application availability and provides a single point of access for users.

The Internet Gateway attached to the VPC enables communication between the instances in public subnets and the internet. This is essential for users to access the application from outside and for the instances to download updates or external packages.

The private subnets (e.g., 10.0.2.0/24) in each Availability Zone host RDS database instances with multi AZ deployment. These databases are not accessible from the internet and are protected within the private network, enhancing data security. They can only be accessed by the web servers in the public subnets, typically over a secure connection. The use of databases in both Availability Zones ensures high availability and failover capability in case one AZ goes down.

There is NAT Gateway in public subnets in order to make databases be able to go to internet and install updates even so they are in private subnet without exposing them to the internet.

---

## Security Groups

### Load Balancer Security Group (SG-LB)

*Inbound Rules:*

| Type  | Protocol | Port | Source    |
|-------|----------|------|-----------|
| HTTP  | TCP      | 80   | 0.0.0.0/0 |
| HTTPS | TCP      | 443  | 0.0.0.0/0 |

*Outbound Rules:*

| Type        | Protocol | Port | Destination |
|-------------|----------|------|-------------|
| All traffic | All      | All  | 0.0.0.0/0   |

---

### Web Server EC2 Security Group (SG-Web)

*Inbound Rules:*

| Type  | Protocol | Port Range | Source              |
|-------|----------|------------|---------------------|
| HTTP  | TCP      | 80         | SG-LB (Load Balancer SG) |
| HTTPS | TCP      | 443        | SG-LB               |

*Outbound Rules:*

| Type        | Protocol | Port Range | Destination |
|-------------|----------|------------|-------------|
| All traffic | All      | All        | 0.0.0.0/0   |

---

### RDS Database Security Group (SG-DB)

*Inbound Rules:*

| Type            | Protocol | Port Range | Source              |
|-----------------|----------|------------|---------------------|
| MySQL/Aurora    | TCP      | 3306       | SG-Web (Web Server SG) |

*Outbound Rules:*

| Type        | Protocol | Port Range | Destination |
|-------------|----------|------------|-------------|
| All traffic | All      | All        | 0.0.0.0/0   |

---

## Route Tables

### 1. Public Route Table

*Associated with:* Public Subnets

| Destination | Target         |
|-------------|----------------|
| 10.0.0.0/16 | local          |
| 0.0.0.0/0   | Internet Gateway |

---

### 2. Private Route Tables AZ-a and AZ-b

*Associated with:* Private Subnets

| Destination | Target     |
|-------------|------------|
| 10.0.0.0/16 | local      |
| 0.0.0.0/0   | NAT Gateway |
