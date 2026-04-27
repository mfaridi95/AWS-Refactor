# AWS-Refactor
Re-architected a multi-tier Java application on AWS using Elastic Beanstalk, RDS, ElastiCache, and Amazon MQ — replacing manual EC2-managed services with fully managed PaaS/SaaS to reduce operational overhead and improve scalability.

This project follows the Re-Architecting / Refactoring strategy — moving away from Lift & Shift towards a cloud-optimized, highly available, and auto-scaling architecture.

```
### Architecture Diagram

User
  ↓
Amazon Route 53 (DNS resolution)
  ↓
Amazon CloudFront (CDN — global edge caching)
  ↓
Application Load Balancer (part of Elastic Beanstalk)
  ↓
EC2 Instances in Auto Scaling Group (Tomcat — managed by Beanstalk)
  ↓                    ↓                    ↓
Amazon RDS         ElastiCache          Amazon MQ
(MySQL DB)        (Memcached Cache)    (Message Broker)
```
**CloudWatch Alarms** monitor the Auto Scaling Group and trigger scale-out/scale-in based on load.
**S3 Bucket** stores build artifacts — deployed to Beanstalk with one click.


```
### Tech Stack

| Layer | Old Setup (Lift & Shift) | New Setup (Refactored) |
|---|---|---|
| App Server | Tomcat on EC2 | Elastic Beanstalk (manages EC2, ALB, ASG) |
| Database | MySQL on EC2 | Amazon RDS (managed, auto-backup) |
| Cache | Memcached on EC2 | Amazon ElastiCache |
| Message Broker | RabbitMQ on EC2 | Amazon MQ (ActiveMQ) |
| DNS | GoDaddy / manual | Amazon Route 53 |
| Content Delivery | None | Amazon CloudFront (CDN) |
| Artifact Storage | Manual copy | Amazon S3 |
| Monitoring | Manual | Amazon CloudWatch Alarms |
| Build Tool | Maven | Maven (`mvn install`) |
| Language | Java | Java |
| OS | Linux | Linux (managed by Beanstalk) |
```
```
### Security Implementations

**1. Security Groups (Layered network access control)**
- Created a dedicated **backend security group** for RDS, ElastiCache, and Amazon MQ
- Only allowed inbound traffic from the **Beanstalk instance security group** — no direct public access to backend services
- Enabled **internal traffic** within the backend security group so services can communicate with each other
- Removed Beanstalk-to-backend rule before teardown to ensure clean deletion

**2. HTTPS / SSL Encryption**
- Added **HTTPS (port 443) listener** on the Elastic Load Balancer
- Configured **SSL certificate via AWS Certificate Manager (ACM)** for encrypted end-to-end communication
- CloudFront distribution configured with SSL — all traffic served over `https://`

**3. CloudFront Security**
- All user traffic routed through **CloudFront** — origin (Load Balancer) is not directly exposed
- CloudFront acts as a protective layer between public internet and application infrastructure
- Content cached at edge locations — reduces direct hits to backend

**4. IAM (Identity & Access Management)**
- Created **IAM role** with S3 full access attached to EC2 instances (Beanstalk) — no hardcoded credentials
- Created **IAM user** with access keys for local machine to push artifacts to S3 via AWS CLI

**5. Private Backend Services**
- RDS, ElastiCache, and Amazon MQ are **not publicly accessible**
- All backend services live inside the backend security group — only reachable from the application tier

**6. DNS Validation**
- Traffic routing verified using **browser DevTools (F12)** → Headers → `via: cloudfront.net` confirms requests route through CloudFront before hitting the load balancer
```
```
### Key Highlights
- Eliminated need for multiple ops teams by using fully managed AWS services
- Achieved **auto-scaling** and **high availability** out of the box via Elastic Beanstalk
- Global audience served via **CloudFront edge locations** — reduced latency worldwide
- **One-click artifact deployment** to Beanstalk from S3
- Infrastructure follows **pay-as-you-go** model — no upfront capital expenditure
- Validated CloudFront routing using browser inspect tools confirming `via: cloudfront.net` in response headers
```


