# AWS Microservices Landing Zone - Terraform + Terragrunt

Repositorio de ejemplo para crear una landing zone / golden path de microservicios Java en AWS usando Terraform para los módulos y Terragrunt para la composición por ambiente.

## Qué contiene

- `modules/`: módulos Terraform reutilizables para networking, EKS, RDS PostgreSQL, DocumentDB, Redis, MSK Kafka, ECR, IAM, KMS, observabilidad, Secrets Manager y WAF.
- `live/`: despliegues reales por ambiente (`dev`, `qa`, `prod`) usando Terragrunt.
- `live/terragrunt.hcl`: configuración común heredada: backend S3, lock DynamoDB, provider AWS y tags comunes.
- `live/<env>/env.hcl`: parámetros específicos del ambiente.
- `live/<env>/<component>/terragrunt.hcl`: dependencias e inputs del componente.

## Orden de despliegue

Terragrunt puede resolver dependencias con `dependency`:

```text
foundation -> network -> platform -> observability
foundation -> network -> data
```

Ejemplo:

```bash
cd terraform/live/dev
terragrunt run-all plan
terragrunt run-all apply
```

O por componente:

```bash
cd terraform/live/dev/foundation
terragrunt apply

cd ../network
terragrunt apply

cd ../platform
terragrunt apply

cd ../data
terragrunt apply

cd ../observability
terragrunt apply
```

## Cambios con terragrunt

Terragrunt se usa donde aporta más valor: en `live/`, para evitar repetir backend/provider/tags, centralizar variables por ambiente y pasar outputs entre capas mediante `dependency`.

Los módulos siguen siendo Terraform puro para poder reutilizarlos desde cualquier pipeline o incluso sin Terragrunt.

## Antes de ejecutar

1. Crea el bucket S3 de state por ambiente o ajusta `state_bucket` en cada `env.hcl`.
2. Crea la tabla DynamoDB `terraform-locks` o ajusta `lock_table`.
3. Cambia los `account_id` placeholder.
4. Revisa tamaños y parámetros productivos antes de aplicar en AWS real.


- 
## Landing Zone
```text
├── Foundation
│   ├── Organizations
│   ├── Control Tower
│   ├── IAM Identity Center
│   └── SCPs
├── Network
│   ├── VPCs
│   ├── Subnets
│   ├── Transit Gateway
│   ├── VPC Endpoints
│   └── DNS
├── Security
│   ├── KMS
│   ├── CloudTrail
│   ├── GuardDuty
│   ├── Security Hub
│   └── Secrets Manager
├── Platform
│   ├── EKS
│   ├── ECR
│   ├── Ingress
│   ├── Service Mesh
│   └── GitOps
├── Data
│   ├── RDS PostgreSQL
│   ├── DocumentDB
│   ├── ElastiCache Redis
│   └── MSK Kafka
├── Observability
│   ├── Logs
│   ├── Metrics
│   ├── Traces
│   └── Alerts
└── Governance
    ├── Tagging
    ├── Cost Management
    ├── Backup
    ├── DR
    └── Compliance
```
