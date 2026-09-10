# Terraform — Fundamentos, instalación y workflow

## Qué es Terraform

Terraform es una herramienta de **Infrastructure as Code (IaC)** que permite describir infraestructura mediante archivos declarativos escritos en HCL (HashiCorp Configuration Language).

En lugar de crear recursos manualmente desde una consola web, se define el estado deseado en código y Terraform calcula qué cambios necesita realizar para alcanzarlo.

Flujo conceptual:

```text
Código Terraform (.tf)
        ↓
terraform init
        ↓
terraform plan
        ↓
Revisión del plan
        ↓
terraform apply
        ↓
Infraestructura real + state
```

El flujo base recomendado es `init → plan → apply`. `plan` permite revisar los cambios antes de ejecutarlos y `apply` materializa el plan aprobado.

## Instalación y verificación

Documentación oficial:

- <https://developer.hashicorp.com/terraform/install>
- <https://developer.hashicorp.com/terraform/cli>

Verificar instalación:

```bash
terraform version
```

Mostrar ayuda:

```bash
terraform -help
terraform plan -help
terraform apply -help
```

## Estructura mínima de un proyecto

```text
infra/
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── terraform.tfvars
└── .gitignore
```

No todos los archivos son obligatorios. Terraform carga los archivos `.tf` del directorio como una única configuración lógica.

## Configuración básica

```hcl
terraform {
  required_version = ">= 1.10.0"

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

> No fijes versiones sin criterio. Definí rangos compatibles y versioná `.terraform.lock.hcl` para mantener builds reproducibles.

## `terraform init`

Inicializa el directorio de trabajo, descarga providers y módulos y configura el backend.

```bash
terraform init
```

Actualizar providers dentro de las restricciones definidas:

```bash
terraform init -upgrade
```

Reconfigurar backend:

```bash
terraform init -reconfigure
```

Migrar state cuando cambia el backend:

```bash
terraform init -migrate-state
```

`terraform init` es idempotente y puede ejecutarse varias veces.

## Formateo y validación

Formatear todos los archivos:

```bash
terraform fmt
```

Comprobar si requieren formato:

```bash
terraform fmt -check -recursive
```

Validar sintaxis y configuración interna:

```bash
terraform validate
```

Una secuencia útil antes de un PR:

```bash
terraform fmt -check -recursive
terraform init -backend=false
terraform validate
```

## `terraform plan`

Genera un execution plan sin modificar infraestructura:

```bash
terraform plan
```

Guardar el plan:

```bash
terraform plan -out=tfplan
```

Aplicar exactamente ese plan:

```bash
terraform apply tfplan
```

Esto es especialmente útil en CI/CD porque separa la etapa de revisión de la etapa de aplicación.

Plan contra un archivo de variables:

```bash
terraform plan -var-file="prod.tfvars"
```

Plan de destrucción:

```bash
terraform plan -destroy
```

Refresh-only:

```bash
terraform plan -refresh-only
```

## `terraform apply`

Aplicar cambios:

```bash
terraform apply
```

Terraform genera un plan y solicita confirmación si no se le pasa un plan previamente guardado.

Aplicar un plan aprobado:

```bash
terraform apply tfplan
```

Para automatizaciones existe:

```bash
terraform apply -auto-approve
```

> `-auto-approve` elimina la confirmación interactiva. Debe utilizarse sólo en pipelines controlados o entornos donde el plan ya haya sido validado.

## `terraform destroy`

Destruye los recursos administrados por la configuración:

```bash
terraform destroy
```

Con variables específicas:

```bash
terraform destroy -var-file="dev.tfvars"
```

> Es uno de los comandos de mayor impacto. En producción conviene exigir revisión y controles adicionales.

## Variables

`variables.tf`:

```hcl
variable "aws_region" {
  description = "Región AWS"
  type        = string
  default     = "us-east-1"
}

variable "environment" {
  description = "Ambiente"
  type        = string
}
```

`terraform.tfvars`:

```hcl
aws_region  = "us-east-1"
environment = "dev"
```

También se pueden pasar variables por CLI:

```bash
terraform plan -var="environment=dev"
```

O mediante variables de entorno:

```bash
export TF_VAR_environment=dev
```

## Outputs

```hcl
output "bucket_name" {
  description = "Nombre del bucket"
  value       = aws_s3_bucket.app.bucket
}
```

Consultar outputs:

```bash
terraform output
terraform output bucket_name
terraform output -json
```

## Locals

```hcl
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

Uso:

```hcl
tags = local.common_tags
```

## Data sources

Permiten consultar recursos o información existente sin administrarla directamente.

```hcl
data "aws_caller_identity" "current" {}

data "aws_region" "current" {}
```

## Recursos

Ejemplo simple:

```hcl
resource "aws_s3_bucket" "app" {
  bucket = "mi-bucket-ejemplo"

  tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

## Dependencias

Terraform construye un grafo de dependencias a partir de referencias:

```hcl
resource "aws_s3_bucket_versioning" "app" {
  bucket = aws_s3_bucket.app.id

  versioning_configuration {
    status = "Enabled"
  }
}
```

Usá `depends_on` sólo cuando la dependencia no pueda inferirse naturalmente:

```hcl
depends_on = [aws_s3_bucket.app]
```

## Comandos de consulta

```bash
terraform providers
terraform output
terraform show
terraform graph
terraform console
```

Mostrar un plan guardado:

```bash
terraform show tfplan
```

Formato JSON:

```bash
terraform show -json tfplan
```

## Archivos que no deben versionarse

Ejemplo `.gitignore`:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
crash.log
crash.*.log
*.tfplan
*.tfvars
*.tfvars.json
```

No ignores `.terraform.lock.hcl`: normalmente debe versionarse.

Si un `.tfvars` no contiene secretos y el equipo desea versionarlo como ejemplo, puede utilizarse un archivo como:

```text
terraform.tfvars.example
```

## Workflow profesional recomendado

```bash
terraform fmt -check -recursive
terraform init
terraform validate
terraform plan -out=tfplan
terraform show tfplan
terraform apply tfplan
```

En equipos:

1. cambios de infraestructura mediante rama;
2. `fmt` y `validate` en CI;
3. generación de plan;
4. revisión del plan en el Pull Request;
5. aprobación;
6. apply controlado;
7. state remoto y locking;
8. auditoría de cambios.

## Documentación oficial

- Terraform CLI: <https://developer.hashicorp.com/terraform/cli>
- `init`: <https://developer.hashicorp.com/terraform/cli/commands/init>
- `plan`: <https://developer.hashicorp.com/terraform/cli/commands/plan>
- `apply`: <https://developer.hashicorp.com/terraform/cli/commands/apply>
- Lenguaje Terraform: <https://developer.hashicorp.com/terraform/language>
