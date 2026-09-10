# Terraform + AWS — Módulos, seguridad y troubleshooting

## Provider de AWS

Terraform utiliza providers para comunicarse con APIs externas. Para AWS:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

Referencia oficial:

- <https://registry.terraform.io/providers/hashicorp/aws/latest/docs>

## Credenciales AWS

No escribas access keys directamente en archivos `.tf`.

Terraform puede reutilizar los mecanismos estándar del SDK/CLI de AWS, por ejemplo perfiles configurados localmente.

```bash
aws sts get-caller-identity --profile desarrollo
export AWS_PROFILE=desarrollo
terraform plan
```

También puede utilizar roles, credenciales temporales e IAM Identity Center/SSO según el entorno.

Antes de aplicar infraestructura:

```bash
aws sts get-caller-identity
aws configure list
terraform workspace show
terraform plan
```

El objetivo es confirmar **identidad AWS + región + workspace + cambios propuestos**.

## Ejemplo AWS simple

```hcl
resource "aws_s3_bucket" "assets" {
  bucket = var.bucket_name

  tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

resource "aws_s3_bucket_versioning" "assets" {
  bucket = aws_s3_bucket.assets.id

  versioning_configuration {
    status = "Enabled"
  }
}
```

## Data sources de AWS

```hcl
data "aws_caller_identity" "current" {}
data "aws_region" "current" {}
```

Outputs útiles:

```hcl
output "aws_account_id" {
  value = data.aws_caller_identity.current.account_id
}

output "aws_region" {
  value = data.aws_region.current.name
}
```

## Módulos

Los módulos permiten encapsular infraestructura reutilizable.

Estructura:

```text
infra/
├── main.tf
├── variables.tf
├── outputs.tf
└── modules/
    └── s3-bucket/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

Consumir un módulo local:

```hcl
module "assets_bucket" {
  source = "./modules/s3-bucket"

  bucket_name = var.bucket_name
  environment = var.environment
}
```

Módulo del Registry:

```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 6.0"

  name = "app-vpc"
  cidr = "10.0.0.0/16"
}
```

> Fijá una restricción de versión razonable para módulos externos. No dependas de una versión flotante sin control.

## `for_each`

```hcl
variable "buckets" {
  type = set(string)
  default = [
    "assets",
    "logs"
  ]
}

resource "aws_s3_bucket" "this" {
  for_each = var.buckets
  bucket   = "mi-app-${each.value}"
}
```

## `count`

```hcl
resource "aws_instance" "worker" {
  count = var.worker_count

  ami           = var.ami_id
  instance_type = "t3.micro"
}
```

`for_each` suele ser preferible cuando cada instancia posee una identidad estable por clave.

## Condicionales

```hcl
instance_type = var.environment == "prod" ? "t3.medium" : "t3.micro"
```

## Lifecycle

```hcl
resource "aws_s3_bucket" "critical" {
  bucket = var.bucket_name

  lifecycle {
    prevent_destroy = true
  }
}
```

`prevent_destroy` puede proteger recursos críticos frente a destrucciones accidentales, aunque no sustituye controles de acceso ni revisión de planes.

Otros argumentos relevantes:

```hcl
lifecycle {
  create_before_destroy = true
}
```

```hcl
lifecycle {
  ignore_changes = [tags]
}
```

Usá `ignore_changes` con criterio: puede ocultar drift que debería ser corregido.

## Recursos sensibles

Valores sensibles:

```hcl
variable "database_password" {
  type      = string
  sensitive = true
}
```

Output sensible:

```hcl
output "database_password" {
  value     = var.database_password
  sensitive = true
}
```

Marcar un valor como `sensitive` evita que se muestre normalmente en salida, pero **no garantiza que desaparezca del state**.

Para secretos reales preferí servicios especializados como AWS Secrets Manager o SSM Parameter Store y diseñá el acceso mediante IAM.

## Validaciones de variables

```hcl
variable "environment" {
  type = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "environment debe ser dev, staging o prod."
  }
}
```

## Preconditions y postconditions

Terraform permite agregar condiciones sobre recursos y data sources para proteger invariantes importantes.

Esto resulta útil para impedir que una configuración válida sintácticamente provoque una arquitectura inválida para el negocio.

## Troubleshooting

### Error: provider no inicializado

```text
Required plugins are not installed
```

Ejecutar:

```bash
terraform init
```

### Cambió provider, módulo o backend

```bash
terraform init -upgrade
```

o según el caso:

```bash
terraform init -reconfigure
```

### Credenciales AWS incorrectas

Validar primero fuera de Terraform:

```bash
aws sts get-caller-identity
```

Con perfil:

```bash
AWS_PROFILE=desarrollo aws sts get-caller-identity
AWS_PROFILE=desarrollo terraform plan
```

### Región incorrecta

```bash
aws configure get region
```

Revisar también el provider:

```hcl
provider "aws" {
  region = var.aws_region
}
```

### State lock

No ejecutar `force-unlock` automáticamente.

Primero confirmar que no existe otro `plan`/`apply` activo. Sólo después:

```bash
terraform force-unlock <LOCK_ID>
```

### Terraform quiere recrear un recurso inesperadamente

No hagas `apply` hasta entender la causa.

Revisar:

```bash
terraform plan
terraform state show <resource-address>
```

Causas frecuentes:

- atributo `ForceNew` del provider;
- cambio de dirección del recurso;
- `count`/`for_each` modificados;
- recurso recreado manualmente;
- import incorrecto;
- cambio de módulo;
- drift.

### Recurso existe pero Terraform no lo conoce

Considerar import:

```bash
terraform import <address> <id>
```

O un bloque declarativo `import`.

### Recurso está en state pero ya no debe ser administrado

```bash
terraform state rm <address>
```

> Esto no destruye el recurso remoto. Revisá cuidadosamente el plan posterior.

## Comandos útiles para diagnóstico

```bash
terraform version
terraform providers
terraform validate
terraform plan
terraform show
terraform state list
terraform state show <address>
terraform output
```

Logging detallado:

```bash
TF_LOG=DEBUG terraform plan
```

Guardar logs:

```bash
TF_LOG=DEBUG TF_LOG_PATH=terraform.log terraform plan
```

No compartas logs sin revisarlos: pueden incluir datos sensibles.

## Terraform y EKS

Terraform puede administrar la infraestructura del cluster EKS y sus dependencias; `kubectl` administra normalmente recursos dentro del cluster.

Flujo típico:

```text
Terraform
  ├── VPC
  ├── IAM
  ├── EKS control plane
  ├── node groups
  └── addons
        ↓
aws eks update-kubeconfig
        ↓
kubectl
  ├── Deployments
  ├── Services
  ├── ConfigMaps
  └── workloads
```

Después de provisionar EKS:

```bash
aws eks update-kubeconfig \
  --region <region> \
  --name <cluster>

kubectl config current-context
kubectl get nodes
```

Conviene separar claramente las responsabilidades entre infraestructura AWS y workloads Kubernetes para evitar estados difíciles de operar.

## Checklist antes de `apply` en AWS

- [ ] `terraform fmt -check -recursive` pasa.
- [ ] `terraform validate` pasa.
- [ ] `aws sts get-caller-identity` muestra la cuenta correcta.
- [ ] región correcta.
- [ ] workspace/backend correctos.
- [ ] el plan fue revisado completo.
- [ ] no aparecen destrucciones inesperadas.
- [ ] cambios IAM fueron revisados especialmente.
- [ ] secretos no aparecen versionados.
- [ ] existe rollback o estrategia de recuperación para cambios críticos.

## Referencias oficiales

- Terraform AWS Provider: <https://registry.terraform.io/providers/hashicorp/aws/latest/docs>
- Modules: <https://developer.hashicorp.com/terraform/language/modules>
- Expressions: <https://developer.hashicorp.com/terraform/language/expressions>
- Lifecycle: <https://developer.hashicorp.com/terraform/language/meta-arguments/lifecycle>
- Sensitive data: <https://developer.hashicorp.com/terraform/language/manage-sensitive-data>
