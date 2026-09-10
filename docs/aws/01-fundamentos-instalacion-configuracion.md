# 01 — AWS CLI: fundamentos, instalación y configuración

## Qué es AWS CLI

AWS Command Line Interface es la herramienta oficial para interactuar con servicios de AWS desde una terminal, scripts y pipelines de automatización.

La sintaxis general es:

```bash
aws <servicio> <operacion> [parametros] [opciones-globales]
```

Ejemplos:

```bash
aws s3 ls
aws ec2 describe-instances
aws lambda list-functions
aws sts get-caller-identity
```

La guía utiliza AWS CLI **v2**.

---

## Instalación

Documentación oficial:

<https://docs.aws.amazon.com/es_es/cli/latest/userguide/getting-started-install.html>

### Verificar instalación

```bash
aws --version
```

Salida típica:

```text
aws-cli/2.x.x Python/3.x ...
```

### Linux x86_64

Seguí siempre el procedimiento oficial actualizado. De forma general:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Actualizar una instalación existente suele requerir:

```bash
sudo ./aws/install --update
```

### Windows

AWS distribuye un instalador MSI oficial. Luego verificá desde PowerShell:

```powershell
aws --version
```

### macOS

AWS mantiene un instalador oficial `.pkg`. También existen gestores de paquetes, pero para documentación reproducible conviene tomar como fuente de verdad el procedimiento oficial.

---

## Anatomía de un comando

```bash
aws ec2 describe-instances --region us-east-1 --output table
```

Partes:

- `aws`: ejecutable;
- `ec2`: servicio;
- `describe-instances`: operación;
- `--region us-east-1`: parámetro global;
- `--output table`: formato de salida.

---

## Sistema de ayuda integrado

```bash
aws help
aws s3 help
aws ec2 help
aws ec2 describe-instances help
```

Este sistema es muy importante porque la AWS CLI se actualiza constantemente.

Referencia completa:

<https://docs.aws.amazon.com/cli/latest/reference/>

---

## Configuración básica

Ejecutar:

```bash
aws configure
```

El asistente solicita normalmente:

```text
AWS Access Key ID
AWS Secret Access Key
Default region name
Default output format
```

Consultar configuración:

```bash
aws configure list
```

Listar perfiles:

```bash
aws configure list-profiles
```

Ver un valor:

```bash
aws configure get region
aws configure get region --profile desarrollo
```

Asignar valores:

```bash
aws configure set region us-east-1
aws configure set output json
```

Por perfil:

```bash
aws configure set region us-east-1 --profile desarrollo
```

---

## Archivos de configuración

En Linux/macOS se utilizan normalmente:

```text
~/.aws/config
~/.aws/credentials
```

En Windows:

```text
%UserProfile%\.aws\config
%UserProfile%\.aws\credentials
```

Ejemplo de `config`:

```ini
[default]
region = us-east-1
output = json

[profile desarrollo]
region = us-east-2
output = table
```

Ejemplo de `credentials`:

```ini
[default]
aws_access_key_id = EJEMPLO
aws_secret_access_key = EJEMPLO
```

Nunca publiques credenciales reales en Git.

---

## Validar quién sos antes de operar

Uno de los comandos más importantes:

```bash
aws sts get-caller-identity
```

Devuelve información como:

```json
{
  "UserId": "...",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/example"
}
```

Con perfil:

```bash
aws sts get-caller-identity --profile produccion
```

Antes de una operación crítica verificá siempre:

```bash
aws sts get-caller-identity
aws configure get region
```

---

## Región

Especificar por comando:

```bash
aws ec2 describe-instances --region us-east-1
```

Variable de entorno:

```bash
export AWS_REGION=us-east-1
```

PowerShell:

```powershell
$env:AWS_REGION="us-east-1"
```

La región importa porque muchos recursos de AWS son regionales.

---

## Formatos de salida

```bash
aws ec2 describe-instances --output json
aws ec2 describe-instances --output yaml
aws ec2 describe-instances --output text
aws ec2 describe-instances --output table
```

Para scripts, `json` suele ser la opción más robusta.

Para inspección humana rápida:

```bash
aws ec2 describe-instances --output table
```

---

## Opciones globales frecuentes

```text
--profile
--region
--output
--query
--no-cli-pager
--no-paginate
--debug
--endpoint-url
--cli-connect-timeout
--cli-read-timeout
```

Ejemplo:

```bash
aws ec2 describe-instances \
  --profile desarrollo \
  --region us-east-1 \
  --output table \
  --no-cli-pager
```

---

## Desactivar el pager

AWS CLI v2 puede utilizar un pager para salidas largas.

Para una ejecución:

```bash
aws ec2 describe-instances --no-cli-pager
```

Configuración global:

```bash
aws configure set cli_pager ""
```

---

## Autocompletado y descubrimiento

La ayuda integrada es la primera herramienta de descubrimiento:

```bash
aws help
aws eks help
aws eks describe-cluster help
```

AWS también ofrece asistentes para ciertas operaciones:

```bash
aws dynamodb wizard new-table
```

La disponibilidad de wizards depende del servicio y de la versión instalada.

---

## Seguridad mínima desde el primer día

No utilices access keys del usuario root.

Preferí, según el entorno:

- IAM Identity Center / SSO;
- IAM Roles;
- credenciales temporales con STS;
- roles de instancia EC2;
- task roles en ECS;
- IAM Roles for Service Accounts o mecanismos equivalentes en EKS;
- secretos administrados fuera del repositorio.

Nunca hagas esto:

```bash
export AWS_SECRET_ACCESS_KEY="clave-real-que-despues-subis-a-git"
```

si existe riesgo de que el shell history, logs o scripts terminen versionados.

---

## Checklist inicial

```bash
aws --version
aws configure list
aws configure list-profiles
aws sts get-caller-identity
aws configure get region
```

Si estos comandos devuelven lo esperado, la CLI está lista para operar.

---

## Referencias oficiales

- Introducción: <https://docs.aws.amazon.com/es_es/cli/latest/userguide/cli-chap-getting-started.html>
- Instalación: <https://docs.aws.amazon.com/es_es/cli/latest/userguide/getting-started-install.html>
- Uso general: <https://docs.aws.amazon.com/es_es/cli/latest/userguide/cli-chap-using.html>
- Ayuda: <https://docs.aws.amazon.com/es_es/cli/latest/userguide/cli-usage-help.html>
- Referencia completa: <https://docs.aws.amazon.com/cli/latest/reference/>
