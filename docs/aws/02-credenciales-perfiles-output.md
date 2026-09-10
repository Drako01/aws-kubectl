# 02 — AWS CLI: credenciales, perfiles, output y consultas

## Perfiles

Los perfiles permiten trabajar con varias cuentas, roles o entornos sin mezclar configuraciones.

```bash
aws configure --profile desarrollo
aws configure --profile staging
aws configure --profile produccion
```

Listarlos:

```bash
aws configure list-profiles
```

Usar uno por comando:

```bash
aws s3 ls --profile desarrollo
```

O por variable de entorno:

```bash
export AWS_PROFILE=desarrollo
```

PowerShell:

```powershell
$env:AWS_PROFILE="desarrollo"
```

Verificar identidad efectiva:

```bash
aws sts get-caller-identity --profile desarrollo
```

---

## IAM Identity Center / SSO

Para organizaciones modernas, AWS recomienda evitar credenciales estáticas cuando sea posible.

Configurar SSO:

```bash
aws configure sso
```

Iniciar sesión:

```bash
aws sso login --profile empresa-dev
```

Cerrar sesión:

```bash
aws sso logout
```

Validar:

```bash
aws sts get-caller-identity --profile empresa-dev
```

Documentación oficial:

<https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html>

---

## AssumeRole con perfiles

Ejemplo conceptual en `~/.aws/config`:

```ini
[profile base]
region = us-east-1

[profile produccion]
role_arn = arn:aws:iam::123456789012:role/ProductionReadOnly
source_profile = base
region = us-east-1
```

Uso:

```bash
aws sts get-caller-identity --profile produccion
```

La política correcta es otorgar únicamente los permisos necesarios.

---

## Variables de entorno útiles

```bash
export AWS_PROFILE=desarrollo
export AWS_REGION=us-east-1
export AWS_DEFAULT_REGION=us-east-1
export AWS_PAGER=""
```

Para credenciales temporales pueden existir:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
```

No las imprimas en logs ni las guardes en repositorios.

---

## Formatos de salida

### JSON

```bash
aws ec2 describe-instances --output json
```

Adecuado para scripts y procesamiento posterior.

### Table

```bash
aws ec2 describe-instances --output table
```

Adecuado para lectura humana.

### Text

```bash
aws ec2 describe-instances --output text
```

Útil en pipelines simples, aunque requiere cuidado con columnas y espacios.

### YAML

```bash
aws ec2 describe-instances --output yaml
```

Útil para lectura estructurada.

---

## `--query` y JMESPath

AWS CLI integra JMESPath para filtrar la respuesta antes de mostrarla.

### Obtener IDs de instancias EC2

```bash
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].InstanceId' \
  --output text
```

### ID, estado y tipo

```bash
aws ec2 describe-instances \
  --query 'Reservations[].Instances[].{Id:InstanceId,State:State.Name,Type:InstanceType}' \
  --output table
```

### Buckets S3

```bash
aws s3api list-buckets \
  --query 'Buckets[].Name' \
  --output text
```

### Funciones Lambda

```bash
aws lambda list-functions \
  --query 'Functions[].{Name:FunctionName,Runtime:Runtime,Memory:MemorySize}' \
  --output table
```

### Clusters EKS

```bash
aws eks list-clusters --query 'clusters[]' --output text
```

---

## Filtros del servicio vs `--query`

No son equivalentes.

Un filtro soportado por el servicio reduce lo que AWS devuelve:

```bash
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running"
```

Luego `--query` transforma la respuesta recibida:

```bash
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[].Instances[].InstanceId' \
  --output text
```

Cuando sea posible, filtrá primero en el servicio y después proyectá con `--query`.

---

## Generar skeletons

AWS CLI puede mostrar una estructura de entrada orientativa:

```bash
aws ec2 run-instances --generate-cli-skeleton input
```

También puede validar parámetros localmente en ciertos casos:

```bash
aws ec2 run-instances \
  --cli-input-json file://input.json \
  --generate-cli-skeleton output
```

Los skeletons pueden cambiar entre versiones; no deben considerarse un contrato estable de versionado.

---

## Parámetros desde archivos

JSON:

```bash
aws iam create-role --cli-input-json file://role.json
```

Archivos binarios o documentos:

```bash
aws lambda update-function-code \
  --function-name api \
  --zip-file fileb://function.zip
```

Diferencia conceptual:

- `file://`: archivo interpretado como texto;
- `fileb://`: archivo binario.

---

## Paginación

Muchos endpoints son paginados.

La CLI suele paginar automáticamente. Podés deshabilitarlo:

```bash
aws s3api list-objects-v2 --bucket ejemplo --no-paginate
```

No confundas paginación del API con el pager visual de la terminal.

Desactivar pager visual:

```bash
--no-cli-pager
```

---

## Confirmación operativa antes de producción

Una rutina útil:

```bash
aws sts get-caller-identity --profile produccion
aws configure get region --profile produccion
```

Y recién después ejecutar el comando de modificación.

---

## Referencias oficiales

- Configuración: <https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html>
- IAM Identity Center: <https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html>
- Filtrado: <https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-filter.html>
- Formatos de salida: <https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-output-format.html>
- JMESPath: <https://jmespath.org/>
