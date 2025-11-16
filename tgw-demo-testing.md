# Transit Gateway Demo Testing Guide

This guide shows how to test the Transit Gateway connectivity created by the `tgw-cf.yaml` CloudFormation template.

## Cost Information

### VPC Reachability Analyzer
- **$0.10 per analysis**
- One-time charge each time you run "Create and analyze path"
- No ongoing costs

### CloudFormation Template Resources
| Resource | Cost | Notes |
|----------|------|-------|
| Transit Gateway Attachments | ~$0.05/hour × 2 = $72/month | Always charged when attached |
| Data Processing | $0.02/GB | Only when traffic flows through TGW |
| EC2 t3.micro instances | ~$8/month each | Free tier eligible |
| VPCs, Subnets, Route Tables | $0 | No charge |

### Demo Cost Optimization
**For Testing:**
- Deploy → Test → Delete same day = ~$0.25 total
- Reachability Analyzer tests = $0.10 each

**For Persistent Demo:**
- ~$88/month if left running continuously
- Most cost comes from TGW attachments (always charged)

### Recommendation
**Deploy → Test → Delete** for demos:
- Template deploys in ~5 minutes
- Testing takes 2 minutes  
- Cleanup is instant
- **Total demo cost: Under $1** for a few hours

![Transit Gateway Architecture](tgw-arch.png)

When the EC2 instance in VPC-A needs to communicate with the EC2 instance in VPC-B, traffic flows through the TGW attachments and Transit Gateway.

### Important simplifications present in this architecture:
• **Single Availability Zone (AZ)** used here but in production you would want to use multiple AZs for redundancy  
• **Only 2 VPCs** used here but enterprise TGWs are typically meant for more (5+ VPCs) as 2 VPCs could use VPC peering instead  
• **Cross-Region communication** is possible (using TGW peering) though not shown here  
• **Minimal subnets per VPC** (production has public/private tiers)  
• **The diagram does not detail security groups and NACLs**, but the CloudFormation example does

## Overview
This guide shows how to test the Transit Gateway connectivity between VPC-A and VPC-B using AWS VPC Reachability Analyzer.

## What is VPC Reachability Analyzer?
- AWS's built-in network connectivity testing tool
- Tests if traffic can flow between two points without sending actual traffic
- Shows the exact path traffic takes through your network
- Identifies where connectivity fails (if it does)

## Testing with VPC Reachability Analyzer

### AWS Console Steps
1. **Navigate to VPC Console**
   - Go to AWS Console → VPC → Reachability Analyzer

2. **Create Network Path**
   - Click "Create and analyze path"
   - **Source type:** Instances
   - **Source:** Select EC2-Instance-VPC-A
   - **Destination type:** Instances  
   - **Destination:** Select EC2-Instance-VPC-B
   - **Protocol:** TCP

3. **Run Analysis**
   - Click "Create and analyze path"
   - Wait 30-60 seconds for results

4. **Review Results**
   - **Reachable:** Shows path through TGW attachments ✅
   - **Not Reachable:** Shows exactly where it fails ❌

## Expected Results

### Successful Path
When working correctly, you should see:
```
Source: EC2-Instance-VPC-A (10.0.1.10)
  ↓
VPC-A Route Table
  ↓  
Transit Gateway Attachment (VPC-A)
  ↓
Transit Gateway
  ↓
Transit Gateway Attachment (VPC-B)
  ↓
VPC-B Route Table
  ↓
Destination: EC2-Instance-VPC-B (10.1.1.10)
```

### Common Failure Points
- **Security Groups:** Not allowing ICMP traffic
- **Route Tables:** Missing routes to TGW
- **TGW Route Tables:** Not propagating routes
- **NACLs:** Blocking traffic (if modified from defaults)

## Cleanup

To avoid ongoing charges:
```bash
aws cloudformation delete-stack --stack-name tgw-demo
```

**Note:** TGW attachments are charged hourly, so delete promptly after testing to minimize costs.

**Note:** TGW attachments are charged hourly, so delete promptly after testing to minimize costs.

## Troubleshooting

### Reachability Analyzer Shows "Not Reachable"
1. Check security groups allow TCP traffic on port 22
2. Verify route tables have TGW routes
3. Confirm TGW attachments are in "Available" state
4. Check for custom NACLs blocking traffic

### Analysis Fails to Complete
- Ensure you have proper IAM permissions for EC2 and VPC services
- Verify the source and destination resources exist
- Check that instances are in "running" state

## Next Steps

### Follow-up Resources
- **TGW vs VPC Peering Decision Tree** - When to choose each approach based on number of VPCs and requirements
- **Production TGW Checklist** - Multi-AZ deployment, security hardening, and monitoring setup
- **TGW Cost Optimization Guide** - Right-sizing attachments and managing data transfer costs
- **Cross-Region TGW Architecture** - Implementing TGW peering for global connectivity

### Ready for Production?
Consider these additional requirements:
- Multi-Availability Zone deployment for high availability
- Shared services VPC for centralized DNS, Active Directory
- Cross-region connectivity requirements
- Security review and compliance considerations
- Monitoring and alerting strategy

### AWS Resources
- Review [AWS Transit Gateway Best Practices](https://docs.aws.amazon.com/vpc/latest/tgw/tgw-best-design-practices.html)
- Use [AWS Pricing Calculator](https://calculator.aws) for production cost estimates
- Consider AWS Well-Architected Framework review for production deployments
