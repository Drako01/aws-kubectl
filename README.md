# AWS CLI, kubectl & Terraform — Guía Profesional

<p align="center">
  <a href="https://aws.amazon.com/cli/" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/amazonwebservices/amazonwebservices-original-wordmark.svg" alt="Amazon Web Services" width="250" />
  </a>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <a href="https://kubernetes.io/docs/reference/kubectl/" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/kubernetes/kubernetes-plain-wordmark.svg" alt="Kubernetes" width="190" />
  </a>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
  <a href="https://developer.hashicorp.com/terraform" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/terraform/terraform-original-wordmark.svg" alt="Terraform" width="210" />
  </a>
</p>

<p align="center">
  <a href="https://aws.amazon.com/cli/" target="_blank" rel="noreferrer">
    <img src="https://img.shields.io/badge/AWS%20CLI-v2-FF9900?logo=amazonaws&logoColor=white" alt="AWS CLI v2" />
  </a>
  <a href="https://kubernetes.io/docs/reference/kubectl/" target="_blank" rel="noreferrer">
    <img src="https://img.shields.io/badge/kubectl-Kubernetes-326CE5?logo=kubernetes&logoColor=white" alt="kubectl" />
  </a>
  <a href="https://developer.hashicorp.com/terraform" target="_blank" rel="noreferrer">
    <img src="https://img.shields.io/badge/Terraform-IaC-844FBA?logo=terraform&logoColor=white" alt="Terraform" />
  </a>
  <img src="https://img.shields.io/badge/Idioma-Espa%C3%B1ol-informational" alt="Español" />
</p>

<p align="center">
  <strong>Guía práctica en español para administrar AWS, operar Kubernetes y gestionar infraestructura como código con Terraform.</strong>
</p>

<p align="center">
  <strong>Autor:</strong> <a href="https://github.com/Drako01">Alejandro Di Stefano</a>
</p>

---

## Objetivo

Este repositorio está pensado como **guía de aprendizaje, referencia operativa y cheat sheet** para tres herramientas fundamentales de Cloud, DevOps e Infrastructure as Code:

- **AWS CLI v2**, para trabajar con servicios de Amazon Web Services desde la terminal;
- **kubectl**, para consultar, desplegar, modificar, depurar y administrar recursos de Kubernetes;
- **Terraform**, para definir y operar infraestructura declarativa, reproducible y versionable.

No intenta reemplazar la documentación oficial. La idea es ofrecer una capa más práctica: explicar **qué hace cada comando, cuándo usarlo, qué riesgo tiene y cómo encaja en un flujo real**.

> AWS ofrece cientos de servicios y miles de operaciones. Por eso esta guía cubre de forma completa la mecánica de AWS CLI y profundiza en los servicios de uso más frecuente. Para operaciones específicas de cualquier servicio se enlaza siempre la referencia oficial completa.

---

## Contenido

| Área | Capítulo | Contenido |
| --- | --- | --- |
| AWS CLI | [01 — Fundamentos, instalación y configuración](docs/aws/01-fundamentos-instalacion-configuracion.md) | Instalación, sintaxis, ayuda, región, configuración inicial y conceptos básicos |
| AWS CLI | [02 — Credenciales, perfiles, salida y consultas](docs/aws/02-credenciales-perfiles-output.md) | IAM, SSO, perfiles, variables, `--query`, JMESPath, JSON, table y text |
| AWS CLI | [03 — Servicios esenciales](docs/aws/03-servicios-esenciales.md) | STS, IAM, S3, EC2, ECR, ECS, EKS, Lambda, CloudWatch, SSM, RDS, DynamoDB, Route 53, CloudFormation y más |
| AWS CLI | [04 — Automatización, seguridad y troubleshooting](docs/aws/04-automatizacion-seguridad-troubleshooting.md) | Bash/PowerShell, paginación, retries, dry-run, seguridad, debug y errores frecuentes |
| kubectl | [05 — Fundamentos, kubeconfig y contexts](docs/kubectl/01-fundamentos-config-contextos.md) | Instalación, sintaxis, clusters, contexts, namespaces y configuración |
| kubectl | [06 — Recursos y workloads](docs/kubectl/02-recursos-workloads.md) | get, describe, create, apply, edit, patch, delete, deployments, jobs, services, configmaps y secrets |
| kubectl | [07 — Observabilidad, debug y operación](docs/kubectl/03-observabilidad-debug-operacion.md) | logs, exec, port-forward, top, events, rollout, scale, drain, cordon y diagnóstico |
| kubectl | [08 — Referencia de comandos](docs/kubectl/04-referencia-comandos.md) | Índice explicado de los comandos principales de kubectl |
| AWS + Kubernetes | [09 — EKS con AWS CLI y kubectl](docs/eks/01-aws-eks-kubectl.md) | Kubeconfig, acceso, ECR, diagnóstico y flujo de trabajo EKS |
| Terraform | [10 — Fundamentos, instalación y workflow](docs/terraform/01-fundamentos-instalacion-workflow.md) | HCL, providers, init, fmt, validate, plan, apply, destroy, variables y outputs |
| Terraform | [11 — State, backends e importación](docs/terraform/02-state-backends-import.md) | State local/remoto, S3 backend, locking, import, moved blocks, drift y workspaces |
| Terraform + AWS | [12 — AWS, módulos, seguridad y troubleshooting](docs/terraform/03-aws-modulos-seguridad-troubleshooting.md) | AWS Provider, módulos, lifecycle, credenciales, secretos, EKS y diagnóstico |
| Consulta rápida | [Cheat Sheet](docs/cheatsheet.md) | Comandos cotidianos de AWS CLI, kubectl y Terraform |

---

# AWS CLI en 60 segundos

AWS CLI sigue la estructura general:

```bash
aws <servicio> <operacion> [opciones]
```

Ejemplos:

```bash
aws sts get-caller-identity
aws s3 ls
aws ec2 describe-instances
aws lambda list-functions
aws eks list-clusters
```

Para descubrir comandos sin salir de la terminal:

```bash
aws help
aws ec2 help
aws ec2 describe-instances help
```

Comprobar versión:

```bash
aws --version
```

---

## Configuración inicial de AWS CLI

```bash
aws configure
aws configure list
aws configure list-profiles
aws sts get-caller-identity
```

Ejecutar con un perfil concreto:

```bash
aws sts get-caller-identity --profile desarrollo
```

> Evitá utilizar credenciales del usuario raíz. En entornos reales preferí roles, credenciales temporales, IAM Identity Center/SSO y mínimo privilegio.

---

# kubectl en 60 segundos

La estructura conceptual es:

```bash
kubectl <comando> <tipo-de-recurso> [nombre] [opciones]
```

Ejemplos:

```bash
kubectl get pods
kubectl get deployments
kubectl describe pod api-7d9c8f76d8-abc12
kubectl logs api-7d9c8f76d8-abc12
kubectl apply -f deployment.yaml
```

Antes de modificar cualquier cluster:

```bash
kubectl config current-context
kubectl config get-contexts
kubectl cluster-info
```

Este hábito evita una de las fallas operativas más comunes: ejecutar correctamente un comando sobre **el cluster equivocado**.

---

# Terraform en 60 segundos

Terraform trabaja declarativamente: se describe la infraestructura deseada y Terraform calcula cómo llegar a ese estado.

Workflow esencial:

```bash
terraform init
terraform fmt -check -recursive
terraform validate
terraform plan
terraform apply
```

Para equipos y CI/CD es preferible guardar el plan revisado:

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

Antes de aplicar cambios sobre AWS:

```bash
aws sts get-caller-identity
terraform workspace show
terraform plan
```

El `state` es parte crítica del funcionamiento de Terraform. En entornos colaborativos conviene utilizar un backend remoto con controles de acceso y locking.

---

## Flujo combinado AWS + Terraform + EKS + kubectl

```text
Terraform
   │
   ├── VPC / IAM / EKS / Node Groups
   │
   ▼
AWS
   │
   │ aws eks update-kubeconfig
   ▼
kubectl
   │
   ├── Deployments
   ├── Services
   ├── ConfigMaps / Secrets
   └── Workloads Kubernetes
```

Terraform resulta especialmente útil para convertir cambios manuales de infraestructura en cambios **reproducibles, auditables y revisables mediante Pull Requests**.

---

## Comandos que conviene tratar con respeto

### AWS

```bash
aws s3 rm s3://bucket --recursive
aws ec2 terminate-instances --instance-ids i-xxxxxxxx
aws cloudformation delete-stack --stack-name produccion
aws rds delete-db-instance --db-instance-identifier produccion
```

### Kubernetes

```bash
kubectl delete namespace produccion
kubectl delete -f .
kubectl replace --force -f recurso.yaml
kubectl drain <node>
kubectl scale deployment <nombre> --replicas=0
```

### Terraform

```bash
terraform destroy
terraform apply -auto-approve
terraform state rm <address>
terraform force-unlock <lock-id>
```

Un comando válido puede tener un impacto operativo enorme. Antes de operaciones destructivas revisá **cuenta, región, perfil, cluster, context, namespace, workspace, backend y plan**.

---

## Documentación oficial

### AWS CLI

- Guía oficial en español: <https://docs.aws.amazon.com/es_es/cli/latest/userguide/>
- Referencia completa de comandos: <https://docs.aws.amazon.com/cli/latest/reference/>
- Instalación AWS CLI v2: <https://docs.aws.amazon.com/es_es/cli/latest/userguide/getting-started-install.html>
- Ejemplos oficiales: <https://docs.aws.amazon.com/es_es/cli/latest/userguide/cli-chap-code-examples.html>

### Kubernetes / kubectl

- Documentación de kubectl: <https://kubernetes.io/docs/reference/kubectl/>
- Quick Reference oficial: <https://kubernetes.io/docs/reference/kubectl/quick-reference/>
- Documentación en español: <https://kubernetes.io/es/docs/reference/>

> Algunas páginas traducidas de Kubernetes pueden estar por detrás de la versión inglesa. Para comportamiento exacto de una versión reciente, verificá siempre la referencia inglesa correspondiente a tu versión de Kubernetes.

### Terraform

- Documentación: <https://developer.hashicorp.com/terraform>
- Terraform CLI: <https://developer.hashicorp.com/terraform/cli>
- Lenguaje Terraform: <https://developer.hashicorp.com/terraform/language>
- Tutorials oficiales: <https://developer.hashicorp.com/terraform/tutorials>
- AWS Provider: <https://registry.terraform.io/providers/hashicorp/aws/latest/docs>

---

## Convenciones utilizadas

Los valores entre `< >` deben reemplazarse:

```bash
aws s3 ls s3://<bucket>
kubectl get pod <pod>
terraform state show <resource-address>
```

Variables útiles:

```bash
export AWS_PROFILE=desarrollo
export AWS_REGION=us-east-1
export NAMESPACE=mi-app
export TF_VAR_environment=dev
```

Los ejemplos priorizan Bash. Cuando una diferencia de PowerShell sea relevante, se indica explícitamente.

---

## Principios operativos recomendados

1. Confirmar identidad y destino antes de modificar infraestructura.
2. Aplicar mínimo privilegio.
3. No versionar access keys, tokens, kubeconfigs, states ni secretos.
4. Preferir infraestructura declarativa para cambios repetibles.
5. Utilizar `terraform plan`, `--dry-run`, `kubectl diff` o equivalentes antes de cambios relevantes.
6. Registrar cambios críticos y operar producción con mecanismos de revisión.
7. Utilizar state remoto y locking para Terraform colaborativo.
8. Versionar `.terraform.lock.hcl` y controlar versiones de providers/módulos.
9. Conocer el rollback o recuperación antes de ejecutar cambios de riesgo.

---

## Licencia y contribuciones

El contenido está orientado a aprendizaje y referencia técnica. Si encontrás un comando desactualizado, una explicación mejorable o un caso práctico que aporte valor, podés abrir un Issue o Pull Request.

---

**Este repositorio no está afiliado oficialmente con Amazon Web Services, HashiCorp ni The Kubernetes Authors/CNCF.**
