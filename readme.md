Transit gateway architecture simplified for learning

Diagram: tgw-arch.png
CloudFormation .yaml file: 

When the EC2 instance in VPC-A needs to communicate with the EC2 instance in VPC-B, traffic flows 
through the TGW attachments and Transit Gateway.

Important simplifications with this architecture:
• Single Availability Zone (AZ) used here but in production you would want to use multiple AZs for redundancy
• Only 2 VPCs used here but enterprise TGWs are typically meant for more (5+ VPCs) as 2 VPCs could use VPC peering instead
• Cross-Region communication is possible (using TGW peering) though not shown here
• Minimal subnets per VPC (production has public/private tiers)
• The diagram does not detail security groups and NACLs, but the CloudFormation example does
