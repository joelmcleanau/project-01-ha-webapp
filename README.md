## Project 01 – Highly Available Web Application

## Objective
Design and deploy a highly available web application architecture in AWS using:

- Custom VPC
- Public and Private Subnets (Multi-AZ)
- Application Load Balancer
- EC2 Auto Scaling Group
- NAT Gateway
- Secure access via SSM (no SSH)

## Goal
Develop practical Solutions Architect reasoning by building real infrastructure and documenting design decisions.

---

This project documents the design, deployment and reasoning behind a production-style highly available AWS architecture.

## Development Environment
- Device: MacBook Air / Desktop Mac
- Git workflow enabled
- AWS CLI configured (ap-southeast-2)

## Architecture Design – VPC Blueprint (Foundation)

### Region
- ap-southeast-2 (Sydney)

### VPC CIDR
- 10.0.0.0/16

### Availability Zones
- Two Availability Zones (for high availability)

### Subnet Strategy (4 total)

| AZ  | Public Subnet | Private Subnet |
|-----|--------------|---------------|
| AZ1 | 10.0.1.0/24  | 10.0.11.0/24  |
| AZ2 | 10.0.2.0/24  | 10.0.12.0/24  |

### Routing Intent
- Public subnets: `0.0.0.0/0 → Internet Gateway`
- Private subnets: `0.0.0.0/0 → NAT Gateway`

### NAT Strategy
For learning, start with a single NAT Gateway when required.  
In production, one NAT Gateway per AZ is recommended to avoid cross-AZ dependency.

## Network Architecture

VPC: 10.0.0.0/16

AZ1
- Public-AZ1 10.0.1.0/24
- Private-AZ1 10.0.11.0/24

AZ2
- Public-AZ2 10.0.2.0/24
- Private-AZ2 10.0.12.0/24

Public route table:
0.0.0.0/0 → Internet Gateway

Private route table:
0.0.0.0/0 → NAT Gateway (when NAT is present)

## Design Decisions

The VPC uses a /16 CIDR block to allow future growth of the subnet structure if the application expands. This provides a large address space while remaining manageable.

The architecture spans two Availability Zones to provide high availability and resilience. Each zone contains both a public and private subnet. The public subnets host internet-facing components such as load balancers, while the private subnets host application compute that should not be directly accessible from the internet.

Public route tables direct external traffic to the Internet Gateway, while the private subnets remain isolated inside the VPC unless outbound access is provided through a NAT Gateway or VPC endpoints.

## Network Architecture Diagram

```mermaid
flowchart TB

    Internet((Internet))
    IGW[Internet Gateway]

    subgraph VPC["Joel-HA-WebApp-VPC 10.0.0.0/16"]
        subgraph AZ1["Availability Zone 1"]
            Pub1["Public-AZ1<br/>10.0.1.0/24"]
            Priv1["Private-AZ1<br/>10.0.11.0/24"]
        end

        subgraph AZ2["Availability Zone 2"]
            Pub2["Public-AZ2<br/>10.0.2.0/24"]
            Priv2["Private-AZ2<br/>10.0.12.0/24"]
        end
    end

    Internet --> IGW
    IGW --> Pub1
    IGW --> Pub2
    Pub1 --> Priv1
    Pub2 --> Priv2

    ## Web Server Deployment

A public EC2 instance was launched in the `Public-AZ1` subnet to host a temporary web server for development and testing.

The instance was configured with an IAM role allowing access through AWS Systems Manager (SSM), avoiding the need for SSH access.

Nginx was installed and started on the instance to act as a simple HTTP web server.

Once HTTP access was allowed in the security group, the default Nginx welcome page was successfully served over the internet using the instance’s public IP address.

This confirmed the network routing, internet gateway, security group configuration, and EC2 instance were functioning correctly.

### Verification

The web server was tested by accessing the instance’s public IP from a browser, which returned the default Nginx welcome page.

This confirmed that:

- HTTP traffic is allowed through the security group
- Nginx is running correctly on the EC2 instance
- The instance is reachable via the internet gateway