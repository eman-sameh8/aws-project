The system is hosted in a single AWS Region, using two Availability Zones (AZs) to ensure high availability and fault tolerance. A Virtual Private Cloud (VPC) with the CIDR block 10.0.0.0/16 is created to provide a logically isolated network environment.
Each AZ contains:
•	A Public Subnet (e.g., 10.0.1.0/24 and 10.0.3.0/24) that hosts an Auto Scaling Group of EC2 instances which serve as web/application servers. These instances are connected to an Application Load Balancer (ALB) to distribute incoming traffic evenly and improve fault tolerance and responsiveness.
•	A Private Subnet (10.0.2.0/24 and 10.0.4.0/24) that hosts MySQL databases, likely deployed using Amazon RDS, to securely store and manage application data. These databases are isolated from internet access to ensure strong security.
To allow private instances (such as RDS or backend services) to access the internet for software updates or patching without being exposed, a NAT Gateway is deployed in the public subnet. The NAT Gateway allows outbound internet traffic from the private subnets while preventing inbound connections from the internet, maintaining a secure setup.
Security Groups (SGs) are used as virtual firewalls to control inbound and outbound traffic:
•	The EC2 instances' security group allows inbound HTTP/HTTPS traffic from the ALB and allows outbound traffic to the internet.
•	The RDS security group allows inbound connections only from the EC2 instances in the application layer, ensuring that only trusted resources can access the database.
•	Security Groups help enforce the principle of least privilege and ensure that each component only communicates with what it needs to.
An Internet Gateway is attached to the VPC, enabling public subnets (and the NAT Gateway) to communicate with the internet.

