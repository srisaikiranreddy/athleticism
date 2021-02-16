# (Developement work in progress.)
# Athleticism
Athleticism is fully hosted in on AWS platform. It is web based project consisting of 3-tier architecture consisting of presentation layer, bussiness layer and data layer.

## App Architecture
- [PresentationLayerRepoLink](https://github.com/srisaikiranreddy/athleticism-ui.git)

- [BussinessLayerRepoLink](https://github.com/srisaikiranreddy/athleticism-webapi.git)

![app-img](https://github.com/srisaikiranreddy/athleticism/blob/main/img/app.png)


## Network Architecture
[athleticism-vpc cloud formation template](https://github.com/srisaikiranreddy/athleticism/blob/main/scripts/athleticism-vpc.template)

The application consists of 3-tier architecture(1 public layer & 2 private layer). 

- Public tier
  - ApplicationLoadBalancer
  - NAT Gateway
- First Private tier
  - AutoScalingGroup
  - EC2 Instances
- Second Private tier
  - Database

![athleticism-vpc-img](https://github.com/srisaikiranreddy/athleticism/blob/main/img/athleticism-vpc.png)


