# (Development work in progress.) The project is not hosted.
# Athleticism
Athleticism is fully hosted on AWS platform and web-based project.

## Demo
Note: Due to budget & cost of hosting and maintaining the project. I am able to provide only localhost demo.
  - Video: Coming soon
  - Screenshots
    - athleticism-ui-screenshoots
    ![athleticism-ui1](https://github.com/srisaikiranreddy/athleticism/blob/main/img/athleticism-ui2.png)    
    ![athleticism-ui2](https://github.com/srisaikiranreddy/athleticism/blob/main/img/athleticism-ui3.png)
    - athleticism-webapi screenshoots
    ![athleticism-webapi](https://github.com/srisaikiranreddy/athleticism/blob/main/img/athleticism-webapi..png)    
    - athleticism-sqlscripts screenshoots
    ![athleticism-sqlscripts](https://github.com/srisaikiranreddy/athleticism/blob/main/img/postgres-db.png)
## App Architecture 
The application is layered architecture consists 3 tier consisting of presentation layer, business layer and data layer. (Note: Further scope of microservices architecture, make more loosely coupled components communicating with each other.)
- [athleticism-ui](https://github.com/srisaikiranreddy/athleticism-ui.git) : Athleticism UI is presentation layer consisting of GUI. The technology used for developing GUI is Angular. 
- [athleticism-webapi](https://github.com/srisaikiranreddy/athleticism-webapi.git) : Athleticism Web API is bussines layer. It is restful web services. The technology used for developing Web API is .net core and ORM used npgsql.entityframework for connecting to PostgresSQL Server.
- [athleticism-sqlscripts](https://github.com/srisaikiranreddy/athleticism-sqlscripts.git) : Sql Scripts for CRUD operations on PostgresSQL Server. 

![app-img](https://github.com/srisaikiranreddy/athleticism/blob/main/img/app.png)


## Network Architecture
- [athleticism-vpc](https://github.com/srisaikiranreddy/athleticism/blob/main/scripts/athleticism-vpc.template): This is Cloud Formation template, the application consists of 3-tier architecture(1 public layer & 2 private layer). The VPC consists of security groups, NAT Gateways, 6 subnets with two availability zones(2 public subnets, 4 private subnets{2 subnets for ec2 instances, 2 subnets for db}), internet gateway attached to VPC. 

- Public tier
  - ApplicationLoadBalancer
  - NAT Gateway
- First Private tier
  - AutoScalingGroup
  - EC2 Instances
- Second Private tier
  - Database

![athleticism-vpc-img](https://github.com/srisaikiranreddy/athleticism/blob/main/img/athleticism-vpc.png)
