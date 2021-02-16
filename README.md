# (Developement work in progress.)
# Athleticism
Athleticism is fully hosted in on AWS platform. It is web based project consisting of 3-tier architecture consisting of presentation layer, bussiness layer and data layer.

## App Architecture 
The application is layered architecture consists 3 tier consisting of presentation layer, busssiness layer and data layer. (Note: Futher scope of micro services architecture, make more loosely coupled components communicating with each other.)
- [athleticism-ui](https://github.com/srisaikiranreddy/athleticism-ui.git) : Athleticism UI is presentation layer consisting of GUI. The technology used for developing GUI is Angular. 
- [athleticism-webapi](https://github.com/srisaikiranreddy/athleticism-webapi.git) : Athleticism Web API is bussines layer. It is restful web services. The technology used for developing web api is .net core and ORM used npgsql.entityframework for connecting to PostgresSQL Server.
- [athleticism-sqlscripts](https://github.com/srisaikiranreddy/athleticism-sqlscripts.git) : Sql Scripts for CRUD operations on PostgresSQL Server. 

![app-img](https://github.com/srisaikiranreddy/athleticism/blob/main/img/app.png)


## Network Architecture
[athleticism-vpc cloud formation template](https://github.com/srisaikiranreddy/athleticism/blob/main/scripts/athleticism-vpc.template)

The application consists of 3-tier architecture(1 public layer & 2 private layer). The VPC consists of security groups, NAT Gatways, 6 subnets with two avalilablity zones(2 public subnets, 4 private subnets{2 subnets for ec2 instances, 2 subnets for db}), internet gateway attached to VPC. 

- Public tier
  - ApplicationLoadBalancer
  - NAT Gateway
- First Private tier
  - AutoScalingGroup
  - EC2 Instances
- Second Private tier
  - Database

![athleticism-vpc-img](https://github.com/srisaikiranreddy/athleticism/blob/main/img/athleticism-vpc.png)


