# Terraform — State, backends e importación

## Qué es el state

Terraform mantiene un archivo de estado para relacionar los recursos declarados en código con los recursos reales que administra.

Por defecto utiliza:

```text
terraform.tfstate
```

El state no es un simple cache: contiene metadatos esenciales para determinar qué crear, modificar o destruir.

> Nunca edites `terraform.tfstate` manualmente salvo una situación extraordinaria y con conocimiento preciso del impacto.

## State local vs remoto

### Local

Adecuado para laboratorios o proyectos personales simples:

```text
terraform.tfstate
```

Limitaciones:

- dificulta colaboración;
- riesgo de pérdida;
- no ofrece locking distribuido;
- puede contener información sensible.

### Remoto

En equipos y producción conviene utilizar un backend remoto.

Ventajas:

- state centralizado;
- colaboración;
- locking según backend;
- mejor control de acceso;
- menor riesgo de corrupción por ejecuciones concurrentes.

Documentación oficial:

- <https://developer.hashicorp.com/terraform/language/backend>
- <https://developer.hashicorp.com/terraform/language/state>

## Backend S3 para AWS

Ejemplo:

```hcl
terraform {
  backend "s3" {
    bucket  = "empresa-terraform-state"
    key     = "prod/network/terraform.tfstate"
    region  = "us-east-1"
    encrypt = true
  }
}
```

Después de modificar la configuración del backend:

```bash
terraform init -reconfigure
```

Si se está migrando un state existente:

```bash
terraform init -migrate-state
```

> El bucket del backend debe existir antes de inicializar ese backend. Es habitual administrarlo mediante un bootstrap separado.

## Seguridad del state

El state puede contener valores sensibles aunque se hayan marcado como `sensitive` en outputs.

Buenas prácticas:

- cifrado en reposo;
- cifrado en tránsito;
- mínimo privilegio;
- versionado del bucket;
- acceso restringido por IAM;
- no guardar state en Git;
- separar state por ambiente y dominio funcional cuando sea razonable;
- backups y recuperación probados.

## Locking

El locking evita que dos procesos modifiquen simultáneamente el mismo state.

Cuando un backend soporta locking, Terraform intenta bloquear el state durante operaciones que pueden modificarlo.

Si aparece un lock legítimo, no lo fuerces sin verificar primero si existe otra ejecución activa.

Liberación manual:

```bash
terraform force-unlock <LOCK_ID>
```

> `force-unlock` debe usarse sólo después de confirmar que ninguna ejecución válida mantiene el lock.

## Inspeccionar state

Listar recursos administrados:

```bash
terraform state list
```

Mostrar un recurso:

```bash
terraform state show aws_s3_bucket.app
```

## Mover recursos dentro del state

```bash
terraform state mv \
  aws_s3_bucket.old \
  aws_s3_bucket.app
```

Esto cambia la dirección del recurso en el state sin recrear el recurso remoto.

Es útil durante refactors, por ejemplo al mover recursos a módulos.

Antes de ejecutar:

```bash
terraform plan
```

Y después:

```bash
terraform plan
```

El segundo plan debe confirmar que Terraform entiende correctamente la nueva dirección.

## Eliminar una referencia del state

```bash
terraform state rm aws_s3_bucket.app
```

Esto hace que Terraform deje de administrar el recurso, pero **no elimina el recurso real**.

Puede ser útil en migraciones, aunque requiere entender bien las consecuencias.

## Importar infraestructura existente

Terraform puede incorporar recursos existentes a su state.

Sintaxis tradicional:

```bash
terraform import <ADDRESS> <ID>
```

Ejemplo:

```bash
terraform import aws_s3_bucket.logs mi-bucket-existente
```

Luego:

```bash
terraform plan
```

El código debe representar el recurso importado; importar no reemplaza la necesidad de definir la configuración.

## Bloques `import`

Terraform también permite declarar imports en configuración:

```hcl
import {
  to = aws_s3_bucket.logs
  id = "mi-bucket-existente"
}
```

Luego:

```bash
terraform plan
terraform apply
```

Este enfoque es más auditable porque el import forma parte del código revisable.

## `moved` blocks

Para refactors declarativos:

```hcl
moved {
  from = aws_s3_bucket.old
  to   = aws_s3_bucket.app
}
```

Esto documenta el cambio de dirección y evita recreaciones innecesarias.

## Refresh y drift

El **drift** ocurre cuando la infraestructura real cambia fuera de Terraform.

Detectarlo:

```bash
terraform plan
```

Refresh-only:

```bash
terraform plan -refresh-only
```

Aplicar sólo la actualización de state:

```bash
terraform apply -refresh-only
```

No utilices refresh-only como sustituto de corregir drift no autorizado. Primero determiná si el cambio manual debe conservarse o revertirse.

## Workspaces

Listar:

```bash
terraform workspace list
```

Crear:

```bash
terraform workspace new dev
```

Cambiar:

```bash
terraform workspace select dev
```

Ver actual:

```bash
terraform workspace show
```

Los workspaces permiten múltiples instancias de state para una misma configuración, pero no siempre son la mejor estrategia para separar ambientes complejos.

Para infraestructuras grandes suele ser más claro separar ambientes mediante directorios, stacks o configuraciones independientes.

## Comandos peligrosos o sensibles

```bash
terraform state rm ...
terraform state mv ...
terraform force-unlock ...
terraform destroy
terraform apply -auto-approve
```

Antes de utilizarlos:

1. verificar workspace;
2. verificar backend;
3. confirmar cuenta AWS;
4. ejecutar `terraform plan` cuando aplique;
5. realizar backup si el cambio afecta directamente al state.

## Documentación oficial

- State: <https://developer.hashicorp.com/terraform/language/state>
- Backends: <https://developer.hashicorp.com/terraform/language/backend>
- `terraform state`: <https://developer.hashicorp.com/terraform/cli/commands/state>
- Import: <https://developer.hashicorp.com/terraform/language/import>
- Workspaces: <https://developer.hashicorp.com/terraform/cli/workspaces>
