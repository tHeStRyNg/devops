## Requirements Definition for Task 2
#### Task 2 - Deploy on Cloud

We decided that we will deploy the Python backend and React frontend on AWS using CloudFormation, for that we will use a CloudFormation template. 
This template creates an Elastic Beanstalk environment that runs the backend and a load balancer with an auto-scaling group to serve the frontend publically.

### AWS CloudFormation - Template

```
AWSTemplateFormatVersion: 2010-09-09

Parameters:
  EnvironmentName:
    Type: String
    Description: Name of the Elastic Beanstalk environment
    Default: my-app
  InstanceType:
    Type: String
    Description: Instance type for the Elastic Beanstalk environment
    Default: t2.micro
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: Key pair to use for the Elastic Beanstalk environment
  BucketName:
    Type: String
    Description: S3 bucket name to deploy the frontend code

Resources:
  BackendApplication:
    Type: 'AWS::ElasticBeanstalk::Application'
    Properties:
      ApplicationName: !Ref EnvironmentName

  BackendEnvironment:
    Type: 'AWS::ElasticBeanstalk::Environment'
    Properties:
      EnvironmentName: !Ref EnvironmentName
      ApplicationName: !Ref BackendApplication
      SolutionStackName: '64bit Amazon Linux 2 v3.4.1 running Python 3.8'
      Tier:
        Type: 'Standard'
        Name: 'WebServer'
      OptionSettings:
        - Namespace: aws:autoscaling:launchconfiguration
          OptionName: IamInstanceProfile
          Value: aws-elasticbeanstalk-ec2-role
        - Namespace: aws:elasticbeanstalk:environment:process:default
          OptionName: Procfile
          Value: web: gunicorn app:app --log-file -
        - Namespace: aws:elasticbeanstalk:container:python
          OptionName: WSGIPath
          Value: app:app
        - Namespace: aws:elasticbeanstalk:healthreporting:system
          OptionName: SystemType
          Value: enhanced
        - Namespace: aws:elasticbeanstalk:healthreporting:system
          OptionName: EnhancedHealthEnabled
          Value: true

  FrontendBucket:
    Type: 'AWS::S3::Bucket'
    Properties:
      BucketName: !Ref BucketName

  FrontendBucketPolicy:
    Type: 'AWS::S3::BucketPolicy'
    Properties:
      Bucket: !Ref FrontendBucket
      PolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Sid: 'PublicReadGetObject'
            Effect: 'Allow'
            Principal: '*'
            Action: 's3:GetObject'
            Resource: !Join ['', ['arn:aws:s3:::', !Ref FrontendBucket, '/*']]
      DependsOn: FrontendBucket

  FrontendLoadBalancer:
    Type: 'AWS::ElasticLoadBalancingV2::LoadBalancer'
    Properties:
      Name: !Join ['-', [!Ref EnvironmentName, 'load-balancer']]
      Scheme: internet-facing
      Type: application
      IpAddressType: ipv4
      SecurityGroups:
        - !Ref FrontendLoadBalancerSecurityGroup
      Subnets:
        - !Ref PublicSubnet1
        - !Ref PublicSubnet2

  FrontendLoadBalancerSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Security group for frontend load balancer
      VpcId: !Ref VpcId
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp: '0.0.0.0/0'

FrontendTargetGroup:
Type: 'AWS::ElasticLoadBalancingV2::TargetGroup'
Properties:
Name: !Join ['-', [!Ref EnvironmentName, 'target-group']]
Port: 80
Protocol: HTTP
TargetType: ip
VpcId: !Ref VpcId

FrontendAutoScalingGroup:
Type: 'AWS::AutoScaling::AutoScalingGroup'
Properties:
AutoScalingGroupName: !Join ['-', [!Ref EnvironmentName, 'auto-scaling-group']]
VPCZoneIdentifier:
- !Ref PublicSubnet1
- !Ref PublicSubnet2
LaunchTemplate:
LaunchTemplateId: !Ref BackendEnvironmentLaunchTemplate
Version: !GetAtt BackendEnvironmentLaunchTemplate.LatestVersionNumber
TargetGroupARNs:
- !Ref FrontendTargetGroup
MinSize: 1
MaxSize: 5
DesiredCapacity: 2
Tags:
- Key: Name
Value: !Join ['-', [!Ref EnvironmentName, 'auto-scaling-group']]
PropagateAtLaunch: 'true'

BackendEnvironmentLaunchTemplate:
Type: 'AWS::EC2::LaunchTemplate'
Properties:
LaunchTemplateName: !Join ['-', [!Ref EnvironmentName, 'launch-template']]
LaunchTemplateData:
ImageId: 'ami-0e92d08d27c382d9b'
InstanceType: !Ref InstanceType
KeyName: !Ref KeyName
UserData:
Fn::Base64: !Sub |
#!/bin/bash
echo "export BUCKET_NAME=${BucketName}" > /etc/profile.d/myapp.sh
BlockDeviceMappings:
- DeviceName: /dev/xvda
Ebs:
VolumeSize: '8'
NetworkInterfaces:
- AssociatePublicIpAddress: 'true'
DeviceIndex: '0'
GroupSet:
- !Ref BackendSecurityGroup
SubnetId: !Ref PrivateSubnet1

BackendSecurityGroup:
Type: 'AWS::EC2::SecurityGroup'
Properties:
GroupDescription: Security group for backend instances
VpcId: !Ref VpcId
SecurityGroupIngress:
- IpProtocol: tcp
FromPort: '5000'
ToPort: '5000'
SourceSecurityGroupId: !Ref FrontendLoadBalancerSecurityGroup
SecurityGroupEgress:
- IpProtocol: tcp
FromPort: '5000'
ToPort: '5000'
DestinationSecurityGroupId: !Ref FrontendLoadBalancerSecurityGroup

Outputs:
BackendEndpoint:
Description: URL of the backend application
Value: !Sub 'http://${BackendEnvironment.EndpointURL}'
FrontendURL:
Description: URL of the frontend application
Value: !Sub 'http://${FrontendLoadBalancer.DNSName}'
```
