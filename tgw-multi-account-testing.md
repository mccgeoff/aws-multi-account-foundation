# Multi-Account Transit Gateway Demo

This demo showcases AWS multi-account architecture patterns using Transit Gateway for cross-account connectivity. While deployed in a single account for simplicity, it demonstrates the networking and security patterns used in production multi-account environments.

## Multi-Account Architecture Overview

![Multi-Account TGW Architecture](tgw-arch4.drawio.png)

The architecture demonstrates Transit Gateway enabling cross-account VPC connectivity within a single region, with the TGW shared from the Management Account via Resource Access Manager (RAM) to workload accounts.

*Note: Security Account (222222222222) exists in complete enterprise setup but typically hosts compliance/monitoring services rather than workload VPCs, so it's not shown in this TGW connectivity demonstration.*


## Architecture Benefits Demonstrated

> **Production Note**: In production environments, route tables would isolate prod/dev environments - both would connect to shared services rather than directly to each other. This demo shows the technical connectivity capability while maintaining proper security boundaries through route table design.

### 1. **Account Isolation**
- **Management Account**: AWS Organizations, billing, and infrastructure services (TGW)
- **Workload Account A (10.0.0.0/16)**: Application and database resources
- **Workload Account B (10.1.0.0/16)**: Application and database resources  
- **Security Account**: Centralized logging and monitoring (not shown in TGW demo)

### 2. **Network Segmentation**
- Separate CIDR blocks prevent IP conflicts
- Cross-account routing through Transit Gateway
- Security groups enforce different access patterns per environment

### 3. **Resource Tagging Strategy**
- `Account-Role`: Infrastructure, Production, DevTest
- `Data-Classification`: Production, Non-Production
- `Security-Level`: Restrictive, Development
- `Auto-Shutdown`: Cost optimization for non-production

### 4. **Security Posture Differences**
- **Workload Account A**: Demonstrates restrictive security groups and enhanced monitoring
- **Workload Account B**: Shows different access patterns for varied workload requirements

## Real-World Implementation

In a production multi-account setup, this architecture would be deployed using separate CloudFormation templates for each account, with the Management Account creating and sharing the Transit Gateway via RAM, and workload accounts accepting the share to create VPC attachments.

The `tgw-cf-multi-account.yaml` template in this repository demonstrates the complete infrastructure pattern, including multi-account tagging and security group strategies.

## Cost Optimization by Account

| Account Type | Cost Strategy |
|--------------|---------------|
| **Management** | Shared infrastructure costs, Reserved Instances for TGW |
| **Workload A** | Application-specific optimization, monitoring costs |
| **Workload B** | Workload-specific cost controls, right-sizing |
| **Security** | Centralized logging with lifecycle policies |

## Prerequisites

- AWS CLI configured with appropriate permissions
- EC2 key pair for instance access
- Understanding of multi-account networking concepts

## Deployment

Deploy the multi-account simulation:

```bash
aws cloudformation create-stack \
  --stack-name multi-account-tgw-demo \
  --template-body file://tgw-cf-multi-account.yaml \
  --parameters ParameterKey=KeyPairName,ParameterValue=your-key-pair \
  --capabilities CAPABILITY_IAM
```

Monitor deployment:
```bash
aws cloudformation describe-stacks \
  --stack-name multi-account-tgw-demo \
  --query 'Stacks[0].StackStatus'
```

## Testing Cross-Account Connectivity

### 1. VPC Reachability Analyzer Test
- **Source**: Workload-A-Instance (10.0.1.10)
- **Destination**: Workload-B-Instance (10.1.1.10)
- **Expected Result**: Reachable via Transit Gateway

### 2. Security Group Validation
- Workload A instance has restrictive access (only from Workload B CIDR)
- Workload B instance demonstrates different access patterns

### 3. Cross-Account Route Verification
```bash
# Check route tables show cross-account routes
aws ec2 describe-route-tables \
  --filters "Name=tag:Account-Role,Values=Workload-A" \
  --query 'RouteTables[*].Routes[?DestinationCidrBlock==`10.1.0.0/16`]'
```

## Multi-Account Best Practices Demonstrated

### 1. **CIDR Planning**
- Non-overlapping CIDR blocks across accounts
- Reserved space for future expansion
- Consistent subnetting strategy

### 2. **Tagging Strategy**
- Account role identification
- Environment classification
- Cost allocation tags
- Security and compliance tags

### 3. **Security Boundaries**
- Account-level isolation
- Different security group policies per environment
- Cross-account access controls

### 4. **Operational Excellence**
- Clear resource ownership through tagging
- Environment-specific configurations
- Automated cost optimization (auto-shutdown tags)

## Production Deployment Considerations

When implementing this in a real multi-account environment:

1. **AWS Organizations Setup**
   - Create organizational units (OUs) for each account type
   - Apply Service Control Policies (SCPs) for governance

2. **Cross-Account IAM Roles**
   - Roles for CloudFormation cross-account deployments
   - Roles for operational access and troubleshooting

3. **Resource Access Manager (RAM)**
   - Share Transit Gateway from Management account
   - Share Route 53 Resolver rules for DNS

4. **Monitoring and Logging**
   - Centralized CloudTrail in Security account
   - Cross-account CloudWatch dashboards
   - Centralized Config rules for compliance

5. **Cost Management**
   - Separate billing per account
   - Cost allocation tags
   - Reserved Instance sharing across accounts

## Cleanup

```bash
aws cloudformation delete-stack --stack-name multi-account-tgw-demo
```

## Next Steps

To implement this as a true multi-account architecture:

1. Set up AWS Organizations with appropriate OUs
2. Create separate AWS accounts for each role
3. Deploy infrastructure template in Management account
4. Set up RAM sharing for Transit Gateway
5. Deploy workload templates in respective accounts
6. Configure cross-account IAM roles and policies

This demo provides the foundation for understanding multi-account networking patterns that scale to enterprise environments.
