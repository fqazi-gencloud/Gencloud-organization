# Care Capture CDK Architecture Diagram

## System Overview
This is a microservices architecture deployed on AWS using CDK, featuring Node.js and FastAPI applications with supporting infrastructure.

```mermaid
graph TB
    %% External Components
    subgraph "External"
        DEV[Developer]
        GITHUB[GitHub Repository]
        INTERNET[Internet Users]
    end

    %% CI/CD Pipeline
    subgraph "CI/CD Pipeline"
        ACTIONS[GitHub Actions]
        OIDC[AWS OIDC Provider]
    end

    %% AWS Cloud Infrastructure
    subgraph "AWS Cloud"
        %% VPC and Networking
        subgraph "VPC (MainVPC)"
            subgraph "Public Subnets (2 AZs)"
                NAT[NAT Gateway]
                RDS[(PostgreSQL RDS)]
                REDIS[(ElastiCache Redis)]
            end
            
            subgraph "Private Subnets (2 AZs)"
                VPC_CONN[VPC Connector]
            end
        end

        %% Container Registry
        subgraph "ECR Repositories"
            ECR_NODE[nodejs-app ECR]
            ECR_FAST[fastapi-app ECR]
        end

        %% App Runner Services
        subgraph "AWS App Runner"
            NODE_SVC[Node.js Service<br/>Port: 3000]
            FAST_SVC[FastAPI Service<br/>Port: 8000]
        end

        %% Supporting Services
        subgraph "AWS Services"
            SSM[SSM Parameter Store]
            SECRETS[Secrets Manager]
            IAM_ROLE[IAM Roles & Policies]
        end
    end

    %% Connections
    DEV --> GITHUB
    GITHUB --> ACTIONS
    ACTIONS --> OIDC
    OIDC --> IAM_ROLE
    
    ACTIONS --> ECR_NODE
    ACTIONS --> ECR_FAST
    
    ECR_NODE --> NODE_SVC
    ECR_FAST --> FAST_SVC
    
    NODE_SVC --> VPC_CONN
    FAST_SVC --> VPC_CONN
    VPC_CONN --> REDIS
    VPC_CONN --> RDS
    
    NODE_SVC --> SSM
    FAST_SVC --> SSM
    NODE_SVC --> SECRETS
    FAST_SVC --> SECRETS
    
    INTERNET --> NODE_SVC
    INTERNET --> FAST_SVC

    %% Styling
    classDef aws fill:#FF9900,stroke:#232F3E,stroke-width:2px,color:#fff
    classDef service fill:#4CAF50,stroke:#2E7D32,stroke-width:2px,color:#fff
    classDef database fill:#2196F3,stroke:#1565C0,stroke-width:2px,color:#fff
    classDef external fill:#9E9E9E,stroke:#424242,stroke-width:2px,color:#fff
    classDef cicd fill:#FF5722,stroke:#D84315,stroke-width:2px,color:#fff

    class ECR_NODE,ECR_FAST,SSM,SECRETS,IAM_ROLE,OIDC aws
    class NODE_SVC,FAST_SVC,VPC_CONN service
    class RDS,REDIS database
    class DEV,GITHUB,INTERNET external
    class ACTIONS cicd
```

## Stack Dependencies

```mermaid
graph TD
    VPC[VPC Stack<br/>- MainVPC<br/>- Public/Private Subnets<br/>- NAT Gateway] 
    
    ECR[ECR Stack<br/>- nodejs-app repo<br/>- fastapi-app repo]
    
    DB[Database Stack<br/>- PostgreSQL RDS<br/>- Security Groups<br/>- SSM Parameters]
    
    CACHE[Cache Stack<br/>- ElastiCache Redis<br/>- Security Groups<br/>- Parameter Groups]
    
    APP[App Runner Stack<br/>- Node.js Service<br/>- FastAPI Service<br/>- VPC Connector<br/>- IAM Roles]

    VPC --> DB
    VPC --> CACHE
    VPC --> APP
    ECR --> APP
    
    classDef stack fill:#E3F2FD,stroke:#1976D2,stroke-width:2px
    class VPC,ECR,DB,CACHE,APP stack
```

## Component Details

### 1. **VPC Stack**
- **MainVPC**: Multi-AZ VPC with CIDR block
- **Public Subnets**: 2 subnets across AZs (CIDR /24)
- **Private Subnets**: 2 subnets with egress (CIDR /24)
- **NAT Gateway**: Single NAT for cost optimization

### 2. **Database Stack**
- **PostgreSQL RDS**: Version 15, t3.micro instance
- **Storage**: 25GB GP2 with auto-scaling disabled
- **Security**: VPC security group, publicly accessible
- **Credentials**: Auto-generated secrets in Secrets Manager
- **Parameters**: Endpoint/port stored in SSM

### 3. **Cache Stack**
- **ElastiCache Redis**: Version 7.0, t4g.micro node
- **Configuration**: Single node, TLS disabled for dev
- **Network**: Public subnets with security groups
- **Parameters**: Custom parameter group for Redis 7

### 4. **ECR Stack**
- **nodejs-app**: Repository for Node.js application
- **fastapi-app**: Repository for FastAPI application
- **Lifecycle**: Auto-delete images on stack destruction

### 5. **App Runner Stack**
- **Node.js Service**: Port 3000, latest image tag
- **FastAPI Service**: Port 8000, specific image digest
- **VPC Integration**: VPC connector for private resource access
- **Environment Variables**: Redis connection details
- **IAM**: Service roles with SSM/Secrets access

### 6. **CI/CD Pipeline**
- **GitHub Actions**: Automated deployment on develop branch
- **OIDC Authentication**: Secure AWS access without long-lived credentials
- **Multi-stage**: Test → Deploy workflow
- **CDK Operations**: Bootstrap, synth, deploy with approval bypass

## Security Configuration

### Network Security
- **VPC Isolation**: Private subnets for sensitive resources
- **Security Groups**: Restrictive inbound rules
- **NAT Gateway**: Controlled outbound internet access

### Application Security
- **IAM Roles**: Least privilege access for App Runner
- **Secrets Management**: Database credentials in Secrets Manager
- **Parameter Store**: Non-sensitive configuration in SSM

### Development vs Production
- **TLS**: Disabled for Redis in development
- **Public Access**: RDS publicly accessible for development
- **Security Groups**: Permissive rules for development ease

## Resource Naming Convention
- **Environment**: `-dev` suffix for development resources
- **Stack Prefixes**: Consistent naming across stacks
- **Export Names**: Cross-stack references with descriptive names

## Deployment Flow
1. **Code Push**: Developer pushes to develop branch
2. **CI Trigger**: GitHub Actions workflow starts
3. **Authentication**: OIDC assumes AWS role
4. **CDK Operations**: Bootstrap → Synth → Deploy
5. **Stack Deployment**: VPC → ECR → Database → Cache → App Runner
6. **Service Updates**: App Runner pulls latest images from ECR

This architecture provides a scalable, secure foundation for microservices deployment with proper separation of concerns and automated CI/CD pipeline.
