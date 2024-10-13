# **Deploy a Dynamic WordPress Website on AWS**

---

## **Project Motivation, Architecture, and Challenges (Cloud Mac Model)**

## **Motivation**
The motivation behind this project was to host a WordPress website on AWS to achieve high availability and scalability for a dynamic content management system while minimizing operational overhead. The website can deliver dynamic, database-driven content efficiently, securely, and at scale by leveraging AWS services. Key services like EC2, RDS, EFS, and Route 53 ensure that the site can handle varying traffic levels and content demands while maintaining high availability and performance. The primary goal was to create a cloud-native infrastructure with automatic failover and seamless scaling, reducing the need for manual intervention.

## **Architecture**
The project follows a **three-tier architecture** hosted on AWS, structured as follows:
1. **VPC Setup**: A Virtual Private Cloud (VPC) with both public and private subnets to separate the network layers.
2. **Web Layer**: EC2 instances, deployed within private subnets, act as web servers for running the WordPress application.
3. **Database Layer**: Amazon RDS (MySQL) is the managed database solution, ensuring data integrity and minimizing administrative tasks.
4. **Load Balancing**: An Application Load Balancer (ALB) is placed in front of the web servers to distribute incoming traffic across EC2 instances. It ensures even load distribution and provides fault tolerance.
5. **Auto Scaling**: AWS Auto Scaling is configured to dynamically adjust the number of EC2 instances based on traffic load, maintaining the site's performance and scalability.
6. **Storage**: Amazon EFS (Elastic File System) is used for shared file storage across web server instances, allowing seamless scaling without manually replicating files.
7. **DNS and SSL**: Amazon Route 53 manages DNS routing for the domain, while AWS Certificate Manager is used to handle SSL certificates for secure HTTPS communication.

## **Challenges**
1. **EC2 to RDS Secure Connection**: 
   - **Issue**: I initially experienced issues managing the secure connection between the EC2 instances and the RDS MySQL database. Incorrect security group configurations prevented proper communication and blocked SSH access to the EC2 instance.
   - **Solution**: Upon troubleshooting with VPC Flow Logs, I discovered that the subnet mask was incorrectly set (using `0.0.0.0/16` instead of `10.0.0.0/16`) in the SSH rule of the security group handling the communication with the EC2 instances. After modifying the inbound and outbound rules, I successfully connected to the EC2 instances using EC2 Instance Connect for SSH access.

2. **Load Balancer Health Checks**: 
   - **Issue**: The Application Load Balancer's health checks reported the EC2 instances as "Unhealthy" even when operating correctly. This prevented traffic from routing to them.
   - **Solution**: After researching AWS documentation, I realized that the ALB health check was failing due to incorrect status codes in the check parameters. I updated the health check to accept HTTP status codes 200, 301, and 302, which resolved the issue, allowing the load balancer to recognize the instances as healthy correctly.

---
