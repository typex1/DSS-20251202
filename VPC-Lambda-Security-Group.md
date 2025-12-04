Mermaid diagram:

```mermaid
graph TB
    subgraph "AWS Account"
        subgraph "VPC (10.0.0.0/16)"
            subgraph "Private Subnet A (10.0.1.0/24)"
                EC2A[EC2 Instance A<br/>10.0.1.10]
                EC2B[EC2 Instance B<br/>10.0.1.20]
            end
            
            subgraph "Private Subnet B (10.0.2.0/24)"
                EC2C[EC2 Instance C<br/>10.0.2.10]
                HyperplaneENI[Hyperplane ENI<br/>10.0.2.100<br/>🔗 Managed by Lambda Service]
            end
            
            SG1[Security Group: Lambda-SG<br/>📤 Outbound: All traffic<br/>📥 Inbound: None]
            SG2[Security Group: EC2-SG<br/>📤 Outbound: All traffic<br/>📥 Inbound: Port 3306 from Lambda-SG]
        end
        
        subgraph "Lambda Service VPC (AWS Managed)"
            Lambda1[Lambda Function 1<br/>VPC Config: Subnet B + Lambda-SG]
            Lambda2[Lambda Function 2<br/>VPC Config: Subnet B + Lambda-SG]
            Lambda3[Lambda Function 3<br/>VPC Config: Subnet A + Lambda-SG]
        end
    end
    
    %% Relationships
    Lambda1 -.->|Network Tunnel| HyperplaneENI
    Lambda2 -.->|Network Tunnel| HyperplaneENI
    Lambda3 -.->|Creates separate ENI<br/>(different subnet)| EC2A
    
    HyperplaneENI -->|Database Connection<br/>Port 3306| EC2A
    HyperplaneENI -->|Database Connection<br/>Port 3306| EC2B
    HyperplaneENI -->|Database Connection<br/>Port 3306| EC2C
    
    %% Security Group Associations
    HyperplaneENI -.->|Associated with| SG1
    EC2A -.->|Associated with| SG2
    EC2B -.->|Associated with| SG2
    EC2C -.->|Associated with| SG2
    
    %% Styling
    classDef lambdaStyle fill:#ff9900,stroke:#232f3e,stroke-width:2px,color:#fff
    classDef eniStyle fill:#4CAF50,stroke:#232f3e,stroke-width:2px,color:#fff
    classDef ec2Style fill:#2196F3,stroke:#232f3e,stroke-width:2px,color:#fff
    classDef sgStyle fill:#9C27B0,stroke:#232f3e,stroke-width:2px,color:#fff
    classDef vpcStyle fill:#f0f0f0,stroke:#232f3e,stroke-width:3px
    
    class Lambda1,Lambda2,Lambda3 lambdaStyle
    class HyperplaneENI eniStyle
    class EC2A,EC2B,EC2C ec2Style
    class SG1,SG2 sgStyle

```
