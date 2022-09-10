# AWS-VPC-end-point
Creating an AWS VPC end point
Access an AWS service using an interface VPC endpoint
PDF
RSS
You can create an interface VPC endpoint to connect to services powered by AWS PrivateLink, including many AWS services.

For each subnet that you specify from your VPC, we create an endpoint network interface in the subnet and assign it a private IP address from the subnet address range. An endpoint network interface is a requester-managed network interface; you can view it in your AWS account, but you can't manage it yourself.

You are billed for hourly usage and data processing charges. For more information, see Interface endpoint pricing.

Contents

Considerations
Prerequisites
Create a VPC endpoint
Test the VPC endpoint
Considerations
Interface VPC endpoints support traffic only over TCP.

AWS services accept connection requests automatically. The service can't initiate requests to resources in your VPC through the VPC endpoint. The endpoint only returns responses to traffic that was initiated by resources in your VPC.

The DNS names created for VPC endpoints are publicly resolvable. They resolve to the private IP addresses of the endpoint network interfaces for the enabled Availability Zones. The private DNS names are not publicly resolvable.

By default, each interface endpoint can support a bandwidth of up to 10 Gbps per Availability Zone and automatically scales up to 100 Gbps. If your application needs higher throughput per zone, contact AWS Support.

There are quotas on your AWS PrivateLink resources. For more information, see AWS PrivateLink quotas.

Prerequisites
Create a private subnet in your VPC and deploy the resources that will access the AWS service using the VPC endpoint in the private subnet.

To use private DNS, you must enable DNS hostnames and DNS resolution for your VPC. For more information, see View and update DNS attributes in the Amazon VPC User Guide.

The security group for the interface endpoint must allow communication between the endpoint network interface and the resources in your VPC that must communicate with the service. By default, the interface endpoint uses the default security group for the VPC. Alternatively, you can create a security group to control the traffic to the endpoint network interfaces from the resources in the VPC. To ensure that tools such as the AWS CLI can make requests over HTTPS from resources in the VPC to the AWS service, the security group must allow inbound HTTPS traffic.

If your resources are in a subnet with a network ACL, verify that the network ACL allows traffic between the endpoint network interfaces and the resources in the VPC.

Create a VPC endpoint
Use the following procedure to create an interface VPC endpoint that connects to an AWS service.

To create an interface endpoint for an AWS service

Open the Amazon VPC console at https://console.aws.amazon.com/vpc/.

In the navigation pane, choose Endpoints.

Choose Create endpoint.

For Service category, choose AWS services.

For Service name, select the service. For more information, see AWS services that integrate with AWS PrivateLink.

For VPC, select the VPC from which you'll access the AWS service.

To create an interface endpoint for Amazon S3, you must clear Additional settings, Enable DNS name. This is because Amazon S3 does not support private DNS for interface VPC endpoints.

For Subnets, select one subnet per Availability Zone from which you'll access the AWS service.

For Security group, select the security groups to associate with the endpoint network interfaces. The security group rules must allow resources that will use the VPC endpoint to communicate with the AWS service to communicate with the endpoint network interface.

For Policy, select Full access to allow all operations by all principals on all resources over the VPC endpoint. Otherwise, select Custom to attach a VPC endpoint policy that controls the permissions that principals have for performing actions on resources over the VPC endpoint. This option is available only if the service supports VPC endpoint policies. For more information, see VPC endpoint policies.

(Optional) To add a tag, choose Add new tag and enter the tag key and the tag value.

Choose Create endpoint.

To create an interface endpoint using the command line

create-vpc-endpoint (AWS CLI)

New-EC2VpcEndpoint (Tools for Windows PowerShell)

Test the VPC endpoint
After you create the VPC endpoint, verify that it's sending requests from your VPC to the AWS service. For this example, we'll demonstrate how to send a request from an EC2 instance in your private subnet to an AWS service, such as Amazon CloudWatch. This requires an EC2 instance in a public subnet from which you can access the instance in the private subnet.

Requirement

Verify that the VPC with the interface VPC endpoint has both a public subnet and a private subnet.

To test the VPC endpoint

Launch an EC2 instance into the private subnet. Use an AMI that comes with the AWS CLI preinstalled (for example, an AMI for Amazon Linux 2) and add an IAM role that allows the instance to call the AWS service. For example, for Amazon CloudWatch, attach the CloudWatchReadOnlyAccess policy to the IAM role.

Launch an EC2 instance into the public subnet and connect to this instance. From the instance in the public subnet, connect to the instance in the private subnet using its private IP address, using the following command.

$ ssh ec2-user@10.0.0.23

Confirm that the instance in the private subnet does not have connectivity to the internet by pinging a well-known public server. If there is 0% packet loss, the instance has internet access. If there is 100% packet loss, the instance has no internet access. For example, the following command pings the Amazon website one time.

$ ping -c 1 www.amazon.com

Run a describe command for the AWS service from the AWS CLI to confirm connectivity to the service from the instance. For example, for Amazon CloudWatch, run the list-metrics command from the instance in the private subnet.

$ aws cloudwatch list-metrics --namespace AWS/EC2

If you get a response, even a response with empty results, then you are connected to the service using AWS PrivateLink. If the command times out, verify that the instance has an IAM role that allows access to the AWS service.
