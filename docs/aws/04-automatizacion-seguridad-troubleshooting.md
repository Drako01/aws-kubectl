# 04 — AWS CLI: automatización, seguridad y troubleshooting

## Automatización segura

AWS CLI es especialmente útil en scripts y CI/CD. Un script profesional debe fallar de forma visible, validar contexto y evitar exponer secretos.

### Bash

```bash
#!/usr/bin/env bash
set -euo pipefail

PROFILE="${AWS_PROFILE:-desarrollo}"
REGION="${AWS_REGION:-us-east-1}"

aws sts get-caller-identity --profile "$PROFILE" >/dev/null
aws s3 ls --profile "$PROFILE" --region "$REGION"
```

### PowerShell

```powershell
$ErrorActionPreference = "Stop"

$profile = if ($env:AWS_PROFILE) { $env:AWS_PROFILE } else { "desarrollo" }
$region = if ($env:AWS_REGION) { $env:AWS_REGION } else { "us-east-1" }

aws sts get-caller-identity --profile $profile | Out-Null
aws s3 ls --profile $profile --region $region
```

---

## Dry run

Algunas operaciones soportan `--dry-run`.

Ejemplo EC2:

```bash
aws ec2 run-instances \
  --image-id ami-xxxxxxxx \
  --instance-type t3.micro \
  --dry-run
```

Un `DryRunOperation` puede indicar que la autorización sería válida sin crear el recurso.

No todos los comandos soportan este flag.

---

## Idempotencia

En automatización conviene preferir operaciones que puedan ejecutarse repetidamente sin producir estados inconsistentes.

Ejemplos:

- `aws s3 sync` para sincronización;
- `aws cloudformation deploy` para infraestructura declarativa;
- consultar estado antes de crear o modificar;
- utilizar client tokens cuando una API los soporte.

---

## Capturar valores

```bash
INSTANCE_ID=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=api" \
  --query 'Reservations[0].Instances[0].InstanceId' \
  --output text)

echo "$INSTANCE_ID"
```

Validá siempre que no sea `None`, vacío o un valor inesperado antes de usarlo en un comando destructivo.

---

## Esperadores (`wait`)

Varios servicios ofrecen waiters:

```bash
aws ec2 wait instance-running --instance-ids i-xxxxxxxx
aws cloudformation wait stack-create-complete --stack-name mi-stack
```

Descubrilos con:

```bash
aws ec2 wait help
```

Son preferibles a bucles manuales cuando existe un waiter oficial adecuado.

---

## Seguridad

### Identidad

Antes de producción:

```bash
aws sts get-caller-identity
aws configure get region
```

### Mínimo privilegio

Una aplicación o usuario operativo no debería recibir `AdministratorAccess` por comodidad.

### Credenciales temporales

Preferí:

- SSO / IAM Identity Center;
- roles;
- STS;
- perfiles que asumen roles;
- credenciales suministradas por el runtime de AWS.

### No registrar secretos

Evitá comandos que impriman secretos en pipelines compartidos:

```bash
aws secretsmanager get-secret-value --secret-id produccion/db
```

Si necesitás consumir el valor, tratá stdout y logs como información sensible.

### Root account

No uses access keys del usuario root para operación normal.

---

## Errores frecuentes

### `Unable to locate credentials`

Diagnóstico:

```bash
aws configure list
aws configure list-profiles
```

Si usás SSO:

```bash
aws sso login --profile empresa
```

### `ExpiredToken`

La sesión temporal venció. Renovar SSO o credenciales temporales.

### `AccessDenied` / `UnauthorizedOperation`

Primero confirmar identidad:

```bash
aws sts get-caller-identity
```

Luego revisar la acción IAM requerida, resource ARN, permission boundary, SCP y resource policy según corresponda.

### Región equivocada

```bash
aws configure get region
```

O especificar:

```bash
--region us-east-1
```

### Perfil equivocado

```bash
aws sts get-caller-identity --profile <perfil>
```

### Salida paginada o terminal aparentemente bloqueada

```bash
--no-cli-pager
```

O:

```bash
aws configure set cli_pager ""
```

### Endpoint o conectividad

Activar diagnóstico:

```bash
aws sts get-caller-identity --debug
```

`--debug` puede producir mucha información y potencialmente datos sensibles. Usalo con criterio antes de compartir logs.

### Certificados TLS

No conviertas esto en solución permanente:

```bash
--no-verify-ssl
```

Deshabilitar validación TLS reduce seguridad. La corrección real suele ser resolver CA, proxy corporativo o configuración del entorno.

---

## Comandos destructivos: patrón recomendado

Antes:

```bash
aws sts get-caller-identity --profile produccion
aws configure get region --profile produccion
```

Consultar exactamente el objetivo:

```bash
aws ec2 describe-instances --instance-ids i-xxxxxxxx --profile produccion
```

Después ejecutar la acción:

```bash
aws ec2 terminate-instances --instance-ids i-xxxxxxxx --profile produccion
```

La misma lógica aplica a RDS, S3, CloudFormation y otros servicios.

---

## Logging y auditoría

AWS CLI es un cliente. La auditoría real de acciones en una cuenta se apoya principalmente en servicios como AWS CloudTrail.

Cuando una operación no tiene explicación clara, revisá:

- identidad que la ejecutó;
- timestamp;
- región;
- request ID;
- evento de CloudTrail;
- política IAM involucrada.

---

## Buenas prácticas para CI/CD

- usar OIDC cuando el proveedor de CI lo soporte;
- evitar access keys estáticas de larga duración;
- separar roles por ambiente;
- limitar permisos del pipeline;
- no imprimir secretos;
- fijar región explícitamente;
- validar identidad al inicio del job;
- aplicar timeouts;
- detener el job ante errores;
- registrar operaciones relevantes sin registrar secretos.

---

## Referencias

- Troubleshooting: <https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-troubleshooting.html>
- Seguridad IAM: <https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html>
- Referencia CLI: <https://docs.aws.amazon.com/cli/latest/reference/>
