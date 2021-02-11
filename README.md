# Athleticism
Athleticism is fully hosted in on AWS platform. It is web based project consisting of 3-tier architecture. 

## App Architecture
PresentationLayer, BussinessLayer, DataLayer 

## Network Architecture

```json
{
    "AWSTemplateFormatVersion": "2010-09-09",
    "Description": "This CloudFormation Template creates the VPC, subnets, routing, NAT Gateways, security groups and IAM Roles to support Athleticism.",
    "Parameters": {
        "AccessCidr": {
            "AllowedPattern": "^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\\/([0-9]|[1-2][0-9]|3[0-2]))$",
            "Description": "The CIDR IP range that is permitted to SSH to bastion instance. Note - a value of 0.0.0.0/0 will allow access from ANY IP address.",
            "Type": "String",
            "Default": "0.0.0.0/0"
        },
        "AthleticismS3Bucket": {
            "Description": "S3 bucket that contains your Athleticism binary file",
            "Type": "String",
            "Default": "Athleticism-test"
        },
        "UseACMBoolean": {
            "AllowedValues": [
                true,
                false
            ],
            "Default": true,
            "Description": "Specifies whether an SSL certificate should be generated for your domain name using AWS Certificate Manager (ACM).  If one is not generated, HTTP will be used and an SSL certificate can be applied after deployment.",
            "Type": "String"
        },
        "UseRoute53Boolean": {
            "AllowedValues": [
                true,
                false
            ],
            "Default": true,
            "Description": "Specifies whether a record set should be created in Route 53 for your Athleticism domain name.  If not, you will recieve a default Elastic Beanstalk DNS name (e.g. Athleticism.us-east-1.elasticbeanstalk.com).",
            "Type": "String"
        },
        "VPCcidr": {
            "Description": "(optional to change) CIDR IP Range for your Athleticism VPC.",
            "AllowedPattern": "^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\\/([0-9]|[1-2][0-9]|3[0-2]))$",
            "Type": "String",
            "Default": "10.1.0.0/16"
        },
        "p1cidr": {
            "Description": "(optional to change) CIDR IP Range for the public subnet in AZ 'a' of your Athleticism VPC.",
            "AllowedPattern": "^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\\/([0-9]|[1-2][0-9]|3[0-2]))$",
            "Type": "String",
            "Default": "10.1.0.0/24"
        },
        "a1cidr": {
            "Description": "(optional to change) CIDR IP Range for the application subnet in AZ 'a' of your Athleticism VPC.",
            "AllowedPattern": "^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\\/([0-9]|[1-2][0-9]|3[0-2]))$",
            "Type": "String",
            "Default": "10.1.1.0/24"
        },
        "d1cidr": {
            "Description": "(optional to change) CIDR IP Range for the database subnet in AZ 'b' of your Athleticism VPC.",
            "AllowedPattern": "^(([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])\\.){3}([0-9]|[1-9][0-9]|1[0-9]{2}|2[0-4][0-9]|25[0-5])(\\/([0-9]|[1-2][0-9]|3[0-2]))$",
            "Type": "String",
            "Default": "10.1.2.0/24"
        }
    },
    "Conditions": {
        "DeployRoute53": {
            "Fn::Equals": [
                true,
                {
                    "Ref": "UseRoute53Boolean"
                }
            ]
        },
        "DeployACM": {
            "Fn::And": [
                {
                    "Fn::Equals": [
                        true,
                        {
                            "Ref": "UseACMBoolean"
                        }
                    ]
                },
                {
                    "Condition": "DeployRoute53"
                }
            ]
        },
        "NoDeployACM": {
            "Fn::Or": [
                {
                    "Fn::Equals": [
                        false,
                        {
                            "Ref": "UseACMBoolean"
                        }
                    ]
                },
                {
                    "Fn::Equals": [
                        false,
                        {
                            "Ref": "UseRoute53Boolean"
                        }
                    ]
                }
            ]
        }
    },
    "Resources": {
        "vpc27bc775f": {
            "Type": "AWS::EC2::VPC",
            "Properties": {
                "CidrBlock": {
                    "Ref": "VPCcidr"
                },
                "InstanceTenancy": "default",
                "EnableDnsSupport": "true",
                "EnableDnsHostnames": "false",
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "AthleticismVPC"
                    }
                ]
            }
        },
        "igw8d9149f4": {
            "Type": "AWS::EC2::InternetGateway",
            "Properties": {
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "AthleticismIGW"
                    }
                ]
            }
        },
        "gw1": {
            "Type": "AWS::EC2::VPCGatewayAttachment",
            "Properties": {
                "VpcId": {
                    "Ref": "vpc27bc775f"
                },
                "InternetGatewayId": {
                    "Ref": "igw8d9149f4"
                }
            }
        },
        "NAT": {
            "DependsOn": "gw1",
            "Type": "AWS::EC2::NatGateway",
            "Properties": {
                "AllocationId": {
                    "Fn::GetAtt": [
                        "EIP",
                        "AllocationId"
                    ]
                },
                "SubnetId": {
                    "Ref": "subnetd3e13a98"
                }
            }
        },
        "EIP": {
            "Type": "AWS::EC2::EIP",
            "Properties": {
                "Domain": "vpc"
            }
        },
        "subnet7aee3531": {
            "Type": "AWS::EC2::Subnet",
            "Properties": {
                "CidrBlock": {
                    "Ref": "d1cidr"
                },
                "AvailabilityZone": {
                    "Fn::Join": [
                        "",
                        [
                            {
                                "Ref": "AWS::Region"
                            },
                            "a"
                        ]
                    ]
                },
                "VpcId": {
                    "Ref": "vpc27bc775f"
                },
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "AthleticismDataZone"
                    }
                ]
            }
        },
        "subnetbae43ff1": {
            "Type": "AWS::EC2::Subnet",
            "Properties": {
                "CidrBlock": {
                    "Ref": "a1cidr"
                },
                "AvailabilityZone": {
                    "Fn::Join": [
                        "",
                        [
                            {
                                "Ref": "AWS::Region"
                            },
                            "a"
                        ]
                    ]
                },
                "VpcId": {
                    "Ref": "vpc27bc775f"
                },
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "AthleticismAppZone"
                    }
                ]
            }
        },
        "subnetd3e13a98": {
            "Type": "AWS::EC2::Subnet",
            "Properties": {
                "CidrBlock": {
                    "Ref": "p1cidr"
                },
                "AvailabilityZone": {
                    "Fn::Join": [
                        "",
                        [
                            {
                                "Ref": "AWS::Region"
                            },
                            "a"
                        ]
                    ]
                },
                "MapPublicIpOnLaunch": "True",
                "VpcId": {
                    "Ref": "vpc27bc775f"
                },
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "AthleticismPublicZone"
                    }
                ]
            }
        },
        "PublicSGSSL": {
            "Condition": "DeployACM",
            "DependsOn": "EIP",
            "Type": "AWS::EC2::SecurityGroup",
            "Properties": {
                "GroupDescription": "Security Group for Load Balancers",
                "VpcId": {
                    "Ref": "vpc27bc775f"
                },
                "SecurityGroupIngress": [
                    {
                        "IpProtocol": "tcp",
                        "FromPort": "443",
                        "ToPort": "443",
                        "CidrIp": {
                            "Ref": "AccessCidr"
                        }
                    },
                    {
                        "IpProtocol": "tcp",
                        "FromPort": "80",
                        "ToPort": "80",
                        "CidrIp": {
                            "Ref": "AccessCidr"
                        }
                    }
                ],
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "AthleticismPublicSecurityGroup"
                    }
                ]
            }
        },
        "PublicSGNoSSL": {
            "Condition": "NoDeployACM",
            "DependsOn": "EIP",
            "Type": "AWS::EC2::SecurityGroup",
            "Properties": {
                "GroupDescription": "Security Group for Load Balancers",
                "VpcId": {
                    "Ref": "vpc27bc775f"
                },
                "SecurityGroupIngress": [
                    {
                        "IpProtocol": "tcp",
                        "FromPort": "443",
                        "ToPort": "443",
                        "CidrIp": {
                            "Ref": "AccessCidr"
                        }
                    },
                    {
                        "IpProtocol": "tcp",
                        "FromPort": "80",
                        "ToPort": "80",
                        "CidrIp": {
                            "Ref": "AccessCidr"
                        }
                    }
                ],
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "AthleticismPublicSecurityGroup"
                    }
                ]
            }
        },
        "AppSG": {
            "Type": "AWS::EC2::SecurityGroup",
            "Properties": {
                "GroupDescription": "Security Group for Application Servers",
                "VpcId": {
                    "Ref": "vpc27bc775f"
                },
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "AthleticismAppSecurityGroup"
                    }
                ]
            }
        },
        "DataSG": {
            "Type": "AWS::EC2::SecurityGroup",
            "Properties": {
                "GroupDescription": "Security Group for Databases",
                "VpcId": {
                    "Ref": "vpc27bc775f"
                },
                "SecurityGroupIngress": [
                    {
                        "IpProtocol": "tcp",
                        "FromPort": "5432",
                        "ToPort": "5432",
                        "SourceSecurityGroupId": {
                            "Ref": "AppSG"
                        }
                    }
                ],
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "AthleticismDBSecurityGroup"
                    }
                ]
            }
        },
        "Route": {
            "Type": "AWS::EC2::Route",
            "Properties": {
                "RouteTableId": {
                    "Ref": "rtb425bda38"
                },
                "DestinationCidrBlock": "0.0.0.0/0",
                "NatGatewayId": {
                    "Ref": "NAT"
                }
            }
        },
        "rtb425bda38": {
            "Type": "AWS::EC2::RouteTable",
            "Properties": {
                "VpcId": {
                    "Ref": "vpc27bc775f"
                },
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "Private"
                    }
                ]
            }
        },
        "rtb3756d74d": {
            "Type": "AWS::EC2::RouteTable",
            "Properties": {
                "VpcId": {
                    "Ref": "vpc27bc775f"
                },
                "Tags": [
                    {
                        "Key": "Name",
                        "Value": "Public"
                    }
                ]
            }
        },
        "subnetroute2": {
            "Type": "AWS::EC2::SubnetRouteTableAssociation",
            "Properties": {
                "RouteTableId": {
                    "Ref": "rtb3756d74d"
                },
                "SubnetId": {
                    "Ref": "subnetd3e13a98"
                }
            }
        },
        "subnetroute5": {
            "Type": "AWS::EC2::SubnetRouteTableAssociation",
            "Properties": {
                "RouteTableId": {
                    "Ref": "rtb425bda38"
                },
                "SubnetId": {
                    "Ref": "subnet7aee3531"
                }
            }
        },
        "subnetroute6": {
            "Type": "AWS::EC2::SubnetRouteTableAssociation",
            "Properties": {
                "RouteTableId": {
                    "Ref": "rtb425bda38"
                },
                "SubnetId": {
                    "Ref": "subnetbae43ff1"
                }
            }
        },
        "route2": {
            "Type": "AWS::EC2::Route",
            "Properties": {
                "DestinationCidrBlock": "0.0.0.0/0",
                "RouteTableId": {
                    "Ref": "rtb3756d74d"
                },
                "GatewayId": {
                    "Ref": "igw8d9149f4"
                }
            },
            "DependsOn": "gw1"
        },
        "ServiceRole": {
            "Type": "AWS::IAM::Role",
            "Properties": {
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Sid": "",
                            "Effect": "Allow",
                            "Principal": {
                                "Service": "elasticbeanstalk.amazonaws.com"
                            },
                            "Action": "sts:AssumeRole",
                            "Condition": {
                                "StringEquals": {
                                    "sts:ExternalId": "elasticbeanstalk"
                                }
                            }
                        }
                    ]
                },
                "Policies": [
                    {
                        "PolicyName": "root",
                        "PolicyDocument": {
                            "Version": "2012-10-17",
                            "Statement": [
                                {
                                    "Effect": "Allow",
                                    "Action": [
                                        "elasticloadbalancing:DescribeInstanceHealth",
                                        "elasticloadbalancing:DescribeLoadBalancers",
                                        "elasticloadbalancing:DescribeTargetHealth",
                                        "ec2:DescribeInstances",
                                        "ec2:DescribeInstanceStatus",
                                        "ec2:GetConsoleOutput",
                                        "ec2:AssociateAddress",
                                        "ec2:DescribeAddresses",
                                        "ec2:DescribeSecurityGroups",
                                        "sqs:GetQueueAttributes",
                                        "sqs:GetQueueUrl",
                                        "autoscaling:DescribeAutoScalingGroups",
                                        "autoscaling:DescribeAutoScalingInstances",
                                        "autoscaling:DescribeScalingActivities",
                                        "autoscaling:DescribeNotificationConfigurations"
                                    ],
                                    "Resource": [
                                        "*"
                                    ]
                                }
                            ]
                        }
                    }
                ],
                "Path": "/"
            }
        },
        "InstanceProfile": {
            "Type": "AWS::IAM::InstanceProfile",
            "Properties": {
                "Path": "/",
                "Roles": [
                    {
                        "Ref": "InstanceProfileRole"
                    }
                ]
            }
        },
        "InstanceProfileRole": {
            "Type": "AWS::IAM::Role",
            "Properties": {
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Principal": {
                                "Service": [
                                    "ec2.amazonaws.com"
                                ]
                            },
                            "Action": [
                                "sts:AssumeRole"
                            ]
                        }
                    ]
                },
                "ManagedPolicyArns": [
                    "arn:aws:iam::aws:policy/service-role/AmazonEC2RoleforSSM"
                ],
                "Policies": [
                    {
                        "PolicyName": "root",
                        "PolicyDocument": {
                            "Version": "2012-10-17",
                            "Statement": [
                                {
                                    "Sid": "BucketAccess",
                                    "Action": [
                                        "s3:*"
                                    ],
                                    "Effect": "Allow",
                                    "Resource": [
                                        {
                                            "Fn::Join": [
                                                "",
                                                [
                                                    "arn:aws",
                                                    ":s3:::elasticbeanstalk-*-",
                                                    {
                                                        "Ref": "AWS::AccountId"
                                                    }
                                                ]
                                            ]
                                        },
                                        {
                                            "Fn::Join": [
                                                "",
                                                [
                                                    "arn:aws",
                                                    ":s3:::elasticbeanstalk-*-",
                                                    {
                                                        "Ref": "AWS::AccountId"
                                                    },
                                                    "/*"
                                                ]
                                            ]
                                        },
                                        {
                                            "Fn::Join": [
                                                "",
                                                [
                                                    "arn:aws",
                                                    ":s3:::elasticbeanstalk-*-",
                                                    {
                                                        "Ref": "AWS::AccountId"
                                                    },
                                                    "-*"
                                                ]
                                            ]
                                        },
                                        {
                                            "Fn::Join": [
                                                "",
                                                [
                                                    "arn:aws",
                                                    ":s3:::elasticbeanstalk-*-",
                                                    {
                                                        "Ref": "AWS::AccountId"
                                                    },
                                                    "-*/*"
                                                ]
                                            ]
                                        },
                                        {
                                            "Fn::Join": [
                                                "",
                                                [
                                                    "arn:aws",
                                                    ":s3:::cf-templates-*",
                                                    {
                                                        "Ref": "AWS::AccountId"
                                                    }
                                                ]
                                            ]
                                        },
                                        {
                                            "Fn::Join": [
                                                "",
                                                [
                                                    "arn:aws",
                                                    ":s3:::cf-templates-*",
                                                    {
                                                        "Ref": "AWS::AccountId"
                                                    },
                                                    "/*"
                                                ]
                                            ]
                                        },
                                        {
                                            "Fn::Join": [
                                                "",
                                                [
                                                    "arn:aws:s3:::",
                                                    {
                                                        "Ref": "AWS::AccountId"
                                                    },
                                                    "-",
                                                    {
                                                        "Fn::Select": [
                                                            1,
                                                            {
                                                                "Fn::Split": [
                                                                    "-",
                                                                    {
                                                                        "Ref": "subnetd3e13a98"
                                                                    }
                                                                ]
                                                            }
                                                        ]
                                                    },
                                                    "-Athleticismebapp"
                                                ]
                                            ]
                                        },
                                        {
                                            "Fn::Join": [
                                                "",
                                                [
                                                    "arn:aws:s3:::",
                                                    {
                                                        "Ref": "AWS::AccountId"
                                                    },
                                                    "-",
                                                    {
                                                        "Fn::Select": [
                                                            1,
                                                            {
                                                                "Fn::Split": [
                                                                    "-",
                                                                    {
                                                                        "Ref": "subnetd3e13a98"
                                                                    }
                                                                ]
                                                            }
                                                        ]
                                                    },
                                                    "-Athleticismebapp",
                                                    "/*"
                                                ]
                                            ]
                                        }
                                    ]
                                },
                                {
                                    "Sid": "MetricsAccess",
                                    "Action": [
                                        "cloudwatch:PutMetricData",
                                        "logs:CreateLogStream",
                                        "logs:PutLogEvents"
                                    ],
                                    "Effect": "Allow",
                                    "Resource": "*"
                                },
                                {
                                    "Sid": "SSMAccess",
                                    "Action": [
                                        "ssm:PutParameter",
                                        "ssm:GetParameter"
                                    ],
                                    "Effect": "Allow",
                                    "Resource": "arn:aws:ssm:*:*:parameter/Athleticism-salt"
                                },
                                {
                                    "Sid": "EncryptedEBS",
                                    "Action": [
                                        "ec2:CreateVolume",
                                        "ec2:AttachVolume",
                                        "ec2:ModifyInstanceAttribute",
                                        "ec2:Describe*"
                                    ],
                                    "Effect": "Allow",
                                    "Resource": "*"
                                },
                                {
                                    "Sid": "CreateNewEBVersion",
                                    "Action": [
                                        "elasticbeanstalk:RetrieveEnvironmentInfo",
                                        "elasticbeanstalk:DescribeEnvironments",
                                        "elasticbeanstalk:DescribeEvents",
                                        "elasticbeanstalk:DescribeConfigurationOptions",
                                        "elasticbeanstalk:DescribeInstancesHealth",
                                        "elasticbeanstalk:DescribeApplicationVersions",
                                        "elasticbeanstalk:DescribeEnvironmentHealth",
                                        "elasticbeanstalk:DescribeApplications",
                                        "elasticbeanstalk:ListPlatformVersions",
                                        "elasticbeanstalk:DescribeEnvironmentResources",
                                        "elasticbeanstalk:DescribeEnvironmentManagedActions",
                                        "elasticbeanstalk:RequestEnvironmentInfo",
                                        "elasticbeanstalk:DescribeEnvironmentManagedActionHistory",
                                        "elasticbeanstalk:CreateApplicationVersion",
                                        "elasticbeanstalk:ValidateConfigurationSettings",
                                        "elasticbeanstalk:DescribeConfigurationSettings",
                                        "elasticbeanstalk:CheckDNSAvailability",
                                        "elasticbeanstalk:ListAvailableSolutionStacks",
                                        "elasticbeanstalk:DescribePlatformVersion",
                                        "elasticbeanstalk:UpdateEnvironment",
                                        "cloudformation:GetTemplate",
                                        "cloudformation:DescribeStackResources",
                                        "cloudformation:DescribeStackResource",
                                        "cloudformation:UpdateStack",
                                        "cloudformation:DescribeStacks",
                                        "cloudformation:DescribeStackEvents",
                                        "autoscaling:DescribeAutoScalingGroups",
                                        "autoscaling:SuspendProcesses",
                                        "autoscaling:DescribeScalingActivities",
                                        "autoscaling:ResumeProcesses",
                                        "autoscaling:DescribeLaunchConfigurations",
                                        "elasticloadbalancing:DescribeTargetGroups",
                                        "elasticloadbalancing:RegisterTargets",
                                        "elasticloadbalancing:DescribeLoadBalancers"
                                    ],
                                    "Effect": "Allow",
                                    "Resource": "*"
                                }
                            ]
                        }
                    }
                ],
                "Path": "/"
            }
        },
        "EC2Role": {
            "Type": "AWS::IAM::Role",
            "Properties": {
                "ManagedPolicyArns": [
                    "arn:aws:iam::aws:policy/service-role/AmazonEC2RoleforSSM"
                ],
                "AssumeRolePolicyDocument": {
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Principal": {
                                "Service": [
                                    "ec2.amazonaws.com"
                                ]
                            },
                            "Action": [
                                "sts:AssumeRole"
                            ]
                        }
                    ]
                },
                "Path": "/"
            }
        },
        "EC2RolePolicies": {
            "Type": "AWS::IAM::Policy",
            "Properties": {
                "PolicyName": "root",
                "PolicyDocument": {
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Action": [
                                "s3:*",
                                "cloudformation:DescribeStackResources",
                                "cloudformation:DescribeStackResource"
                            ],
                            "Resource": [
                                {
                                    "Fn::Join": [
                                        "",
                                        [
                                            "arn:aws:s3:::",
                                            {
                                                "Ref": "AthleticismS3Bucket"
                                            }
                                        ]
                                    ]
                                },
                                {
                                    "Fn::Join": [
                                        "",
                                        [
                                            "arn:aws:s3:::",
                                            {
                                                "Ref": "AthleticismS3Bucket"
                                            },
                                            "/*"
                                        ]
                                    ]
                                },
                                {
                                    "Fn::Join": [
                                        "",
                                        [
                                            "arn:aws:s3:::",
                                            {
                                                "Ref": "AWS::AccountId"
                                            },
                                            "-",
                                            {
                                                "Fn::Select": [
                                                    1,
                                                    {
                                                        "Fn::Split": [
                                                            "-",
                                                            {
                                                                "Ref": "subnetd3e13a98"
                                                            }
                                                        ]
                                                    }
                                                ]
                                            },
                                            "-Athleticismebapp"
                                        ]
                                    ]
                                },
                                {
                                    "Fn::Join": [
                                        "",
                                        [
                                            "arn:aws:s3:::",
                                            {
                                                "Ref": "AWS::AccountId"
                                            },
                                            "-",
                                            {
                                                "Fn::Select": [
                                                    1,
                                                    {
                                                        "Fn::Split": [
                                                            "-",
                                                            {
                                                                "Ref": "subnetd3e13a98"
                                                            }
                                                        ]
                                                    }
                                                ]
                                            },
                                            "-Athleticismebapp",
                                            "/*"
                                        ]
                                    ]
                                },
                                {
                                    "Fn::Join": [
                                        "",
                                        [
                                            "arn:aws:s3:::",
                                            {
                                                "Ref": "AWS::AccountId"
                                            },
                                            "-",
                                            {
                                                "Fn::Select": [
                                                    1,
                                                    {
                                                        "Fn::Split": [
                                                            "-",
                                                            {
                                                                "Ref": "subnetd3e13a98"
                                                            }
                                                        ]
                                                    }
                                                ]
                                            },
                                            "-Athleticismrepository"
                                        ]
                                    ]
                                },
                                {
                                    "Fn::Join": [
                                        "",
                                        [
                                            "arn:aws:s3:::",
                                            {
                                                "Ref": "AWS::AccountId"
                                            },
                                            "-",
                                            {
                                                "Fn::Select": [
                                                    1,
                                                    {
                                                        "Fn::Split": [
                                                            "-",
                                                            {
                                                                "Ref": "subnetd3e13a98"
                                                            }
                                                        ]
                                                    }
                                                ]
                                            },
                                            "-Athleticismrepository",
                                            "/*"
                                        ]
                                    ]
                                }
                            ]
                        },
                        {
                            "Effect": "Allow",
                            "Action": [
                                "elasticbeanstalk:ListAvailableSolutionStacks"
                            ],
                            "Resource": [
                                "*"
                            ]
                        },
                        {
                            "Effect": "Allow",
                            "Action": [
                                "logs:CreateLogGroup",
                                "logs:CreateLogStream",
                                "logs:PutLogEvents",
                                "logs:DescribeLogStreams"
                            ],
                            "Resource": "arn:aws:logs:*:*:*"
                        }
                    ]
                },
                "Roles": [
                    {
                        "Ref": "EC2Role"
                    }
                ]
            }
        },
        "EC2InstanceProfile": {
            "Type": "AWS::IAM::InstanceProfile",
            "Properties": {
                "Path": "/",
                "Roles": [
                    {
                        "Ref": "EC2Role"
                    }
                ]
            }
        },
        "AthleticismUser": {
            "Type": "AWS::IAM::User",
            "Properties": {
                "Policies": [
                    {
                        "PolicyName": "S3Access",
                        "PolicyDocument": {
                            "Statement": [
                                {
                                    "Effect": "Allow",
                                    "Action": [
                                        "s3:*"
                                    ],
                                    "Resource": [
                                        {
                                            "Fn::Join": [
                                                "",
                                                [
                                                    "arn:aws:s3:::",
                                                    {
                                                        "Ref": "AWS::AccountId"
                                                    },
                                                    "-",
                                                    {
                                                        "Fn::Select": [
                                                            1,
                                                            {
                                                                "Fn::Split": [
                                                                    "-",
                                                                    {
                                                                        "Ref": "subnetd3e13a98"
                                                                    }
                                                                ]
                                                            }
                                                        ]
                                                    },
                                                    "-Athleticismrepository"
                                                ]
                                            ]
                                        },
                                        {
                                            "Fn::Join": [
                                                "",
                                                [
                                                    "arn:aws:s3:::",
                                                    {
                                                        "Ref": "AWS::AccountId"
                                                    },
                                                    "-",
                                                    {
                                                        "Fn::Select": [
                                                            1,
                                                            {
                                                                "Fn::Split": [
                                                                    "-",
                                                                    {
                                                                        "Ref": "subnetd3e13a98"
                                                                    }
                                                                ]
                                                            }
                                                        ]
                                                    },
                                                    "-Athleticismrepository",
                                                    "/*"
                                                ]
                                            ]
                                        }
                                    ]
                                }
                            ]
                        }
                    }
                ]
            }
        },
        "AthleticismAccessKey": {
            "Type": "AWS::IAM::AccessKey",
            "Properties": {
                "UserName": {
                    "Ref": "AthleticismUser"
                }
            }
        }
    },
    "Outputs": {
        "VPCId": {
            "Value": {
                "Ref": "vpc27bc775f"
            }
        },
        "SubnetPublicA": {
            "Value": {
                "Ref": "subnetd3e13a98"
            }
        },
        "SubnetAppA": {
            "Value": {
                "Ref": "subnetbae43ff1"
            }
        },
        "SubnetDataA": {
            "Value": {
                "Ref": "subnet7aee3531"
            }
        },
        "SGPublic": {
            "Value": {
                "Fn::If": [
                    "DeployACM",
                    {
                        "Ref": "PublicSGSSL"
                    },
                    {
                        "Ref": "PublicSGNoSSL"
                    }
                ]
            }
        },
        "SGApp": {
            "Value": {
                "Ref": "AppSG"
            }
        },
        "SGData": {
            "Value": {
                "Ref": "DataSG"
            }
        },
        "EBServiceRole": {
            "Value": {
                "Ref": "ServiceRole"
            }
        },
        "EBInstanceProfile": {
            "Value": {
                "Ref": "InstanceProfile"
            }
        },
        "TempEC2InstanceProfile": {
            "Value": {
                "Ref": "EC2InstanceProfile"
            }
        },
        "S3AccessKey": {
            "Value": {
                "Ref": "AthleticismAccessKey"
            }
        },
        "S3SecretKey": {
            "Value": {
                "Fn::GetAtt": [
                    "AthleticismAccessKey",
                    "SecretAccessKey"
                ]
            }
        }
    }
}

```
