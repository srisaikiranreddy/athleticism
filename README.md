[#f03c15](Developement work in progress.) `#f03c15`
# Athleticism
Athleticism is fully hosted in on AWS platform. It is web based project consisting of 3-tier architecture consisting of presentation layer, bussiness layer and data layer.

## App Architecture
[PresentationLayerRepoLink](https://github.com/srisaikiranreddy/athleticism-ui.git)

[BussinessLayerRepoLink](https://github.com/srisaikiranreddy/athleticism-webapi.git)

![app-img](https://github.com/srisaikiranreddy/athleticism/blob/main/img/app.png)


## Network Architecture
[athleticism-vpc cloud formation template](https://github.com/srisaikiranreddy/athleticism/blob/main/scripts/athleticism-vpc.template)

The application consists of 3-tier architecture(1-public & 2 private). 

Public tier
-aplication load balancer
-NAT Gateway
Private tier

![athleticism-vpc-img](https://github.com/srisaikiranreddy/athleticism/blob/main/img/athleticism-vpc.png)


