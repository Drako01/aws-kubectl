# DevOps Toolkit — Guía Profesional

<p align="center">
  <a href="https://aws.amazon.com/cli/" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/amazonwebservices/amazonwebservices-original-wordmark.svg" alt="Amazon Web Services" width="165" />
  </a>
  &nbsp;&nbsp;&nbsp;
  <a href="https://kubernetes.io/docs/reference/kubectl/" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/kubernetes/kubernetes-plain-wordmark.svg" alt="Kubernetes" width="125" />
  </a>
  &nbsp;&nbsp;&nbsp;
  <a href="https://developer.hashicorp.com/terraform" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/terraform/terraform-original-wordmark.svg" alt="Terraform" width="135" />
  </a>
  &nbsp;&nbsp;&nbsp;
  <a href="https://helm.sh/docs/" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/helm/helm-original.svg" alt="Helm" width="72" />
  </a>
  &nbsp;&nbsp;&nbsp;
  <a href="https://docs.github.com/actions" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/githubactions/githubactions-original.svg" alt="GitHub Actions" width="72" />
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
  <a href="https://helm.sh/docs/" target="_blank" rel="noreferrer">
    <img src="https://img.shields.io/badge/Helm-Kubernetes-0F1689?logo=helm&logoColor=white" alt="Helm" />
  </a>
  <a href="https://docs.github.com/actions" target="_blank" rel="noreferrer">
    <img src="https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-2088FF?logo=githubactions&logoColor=white" alt="GitHub Actions" />
  </a>
  <img src="https://img.shields.io/badge/Idioma-Espa%C3%B1ol-informational" alt="Español" />
</p>

<p align="center">
  <strong>Guía práctica en español para AWS, Kubernetes, Terraform, Helm y pipelines CI/CD.</strong>
</p>

<p align="center">
  <strong>Autor:</strong> <a href="https://github.com/Drako01">Alejandro Di Stefano</a>
</p>

---

## Objetivo

Este repositorio funciona como **guía de aprendizaje, referencia operativa y cheat sheet DevOps**. Reúne herramientas y flujos que normalmente se usan en conjunto en entornos Cloud y Kubernetes:

- **AWS CLI v2**, para operar servicios de Amazon Web Services desde terminal;
- **kubectl**, para administrar recursos y workloads Kubernetes;
- **Terraform**, para Infrastructure as Code;
- **Helm**, para empaquetado, parametrización y releases Kubernetes;
- **Amazon EKS**, desde acceso básico hasta operación avanzada;
- **GitHub Actions**, para ejemplos de CI/CD hacia ECR y EKS;
- **Troubleshooting**, con casos reales y secuencias de diagnóstico.

No intenta reemplazar la documentación oficial. La guía agrega una capa práctica: **qué hace cada herramienta, cuándo usarla, cómo combinarla con las demás y cómo diagnosticar fallas reales**.

---

## Contenido

| Área | Capítulo | Contenido |
| --- | --- | --- |
| AWS CLI | [01 — Fundamentos, instalación y configuración](docs/aws/01-fundamentos-instalacion-configuracion.md) | Instalación, sintaxis, ayuda, región y configuración inicial |
| AWS CLI | [02 — Credenciales, perfiles, salida y consultas](docs/aws/02-credenciales-perfiles-output.md) | IAM, SSO, perfiles, variables, JMESPath y outputs |
| AWS CLI | [03 — Servicios esenciales](docs/aws/03-servicios-esenciales.md) | STS, IAM, S3, EC2, ECR, ECS, EKS, Lambda, CloudWatch, SSM, RDS, DynamoDB y más |
| AWS CLI | [04 — Automatización, seguridad y troubleshooting](docs/aws/04-automatizacion-seguridad-troubleshooting.md) | Bash/PowerShell, retries, dry-run, seguridad y diagnóstico |
| kubectl | [05 — Fundamentos, kubeconfig y contexts](docs/kubectl/01-fundamentos-config-contextos.md) | Clusters, contexts, namespaces y configuración |
| kubectl | [06 — Recursos y workloads](docs/kubectl/02-recursos-workloads.md) | Deployments, Jobs, Services, ConfigMaps, Secrets y storage |
| kubectl | [07 — Observabilidad, debug y operación](docs/kubectl/03-observabilidad-debug-operacion.md) | Logs, exec, port-forward, events, rollout, scale y nodos |
| kubectl | [08 — Referencia de comandos](docs/kubectl/04-referencia-comandos.md) | Índice explicado de los comandos principales |
| EKS | [09 — EKS con AWS CLI y kubectl](docs/eks/01-aws-eks-kubectl.md) | Kubeconfig, acceso, ECR y flujo básico EKS |
| EKS | [10 — Operación avanzada de EKS](docs/eks/02-operacion-avanzada.md) | Access Entries, Node Groups, add-ons, upgrades, Pod Identity y diagnóstico |
| Terraform | [11 — Fundamentos, instalación y workflow](docs/terraform/01-fundamentos-instalacion-workflow.md) | HCL, providers, init, plan, apply, variables y outputs |
| Terraform | [12 — State, backends e importación](docs/terraform/02-state-backends-import.md) | State remoto, locking, import, drift y workspaces |
| Terraform + AWS | [13 — AWS, módulos, seguridad y troubleshooting](docs/terraform/03-aws-modulos-seguridad-troubleshooting.md) | AWS Provider, módulos, lifecycle, secretos y EKS |
| Helm | [14 — Helm: charts y operación profesional](docs/helm/01-fundamentos-charts-operacion.md) | Charts, releases, repos, install, upgrade, rollback, OCI y buenas prácticas |
| Troubleshooting | [15 — Casos reales](docs/troubleshooting/01-casos-reales.md) | Unauthorized, Forbidden, CrashLoopBackOff, ImagePullBackOff, Pending, DNS, Terraform, Helm y EKS |
| CI/CD | [16 — GitHub Actions + Kubernetes](docs/cicd/01-github-actions-kubernetes.md) | OIDC, ECR, EKS, kubectl, Helm, ambientes, rollback y seguridad |
| Consulta rápida | [Cheat Sheet](docs/cheatsheet.md) | AWS CLI, kubectl, Terraform y Helm |

---

# Stack DevOps de referencia

```text
GitHub
  │
  ▼
GitHub Actions
  │
  ├── test / lint / build
  ├── Terraform plan/apply
  ├── Docker build
  └── push image
          │
          ▼
         ECR
          │
          ▼
         EKS
          │
     ┌────┴────┐
     ▼         ▼
  kubectl     Helm
     │         │
     └────┬────┘
          ▼
   Kubernetes workloads
```

---

# AWS CLI en 60 segundos

```bash
aws sts get-caller-identity
aws s3 ls
aws ec2 describe-instances
aws eks list-clusters
```

Configuración:

```bash
aws configure
aws configure list
aws configure list-profiles
```

---

# kubectl en 60 segundos

```bash
kubectl config current-context
kubectl get nodes
kubectl get pods -A
kubectl describe pod <pod>
kubectl logs <pod>
kubectl apply -f deployment.yaml
```

Antes de modificar un cluster:

```bash
kubectl config current-context
kubectl config view --minify
```

---

# Terraform en 60 segundos

```bash
terraform init
terraform fmt -check -recursive
terraform validate
terraform plan
terraform apply
```

Para CI/CD y cambios revisados:

```bash
terraform plan -out=tfplan
terraform apply tfplan
```

---

# Helm en 60 segundos

```bash
helm repo update
helm lint ./chart
helm template mi-app ./chart
helm upgrade --install mi-app ./chart \
  --namespace mi-app \
  --create-namespace \
  --wait
```

Rollback:

```bash
helm history mi-app -n mi-app
helm rollback mi-app <revision> -n mi-app
```

---

## Flujo combinado AWS + Terraform + EKS + Helm/kubectl

```text
Terraform
   │
   ├── VPC / IAM / EKS / Node Groups
   ▼
AWS / EKS
   │
   ├── aws eks update-kubeconfig
   ▼
Kubernetes
   │
   ├── kubectl
   └── Helm
```

Terraform gestiona infraestructura; `kubectl` y Helm operan recursos Kubernetes. Mantener esa separación ayuda a evitar estados duplicados y ownership ambiguo.

---

## Comandos que conviene tratar con respeto

### AWS

```bash
aws s3 rm s3://bucket --recursive
aws ec2 terminate-instances --instance-ids i-xxxxxxxx
```

### Kubernetes

```bash
kubectl delete namespace produccion
kubectl drain <node>
kubectl scale deployment <nombre> --replicas=0
```

### Terraform

```bash
terraform destroy
terraform state rm <address>
terraform force-unlock <lock-id>
```

### Helm

```bash
helm uninstall <release>
helm rollback <release> <revision>
helm upgrade <release> <chart> --force
```

Antes de operaciones de riesgo revisá **cuenta, región, perfil, cluster, context, namespace, workspace, backend, release y plan**.

---

## Documentación oficial

### AWS CLI

- <https://docs.aws.amazon.com/es_es/cli/latest/userguide/>
- <https://docs.aws.amazon.com/cli/latest/reference/>

### Kubernetes / kubectl

- <https://kubernetes.io/docs/reference/kubectl/>
- <https://kubernetes.io/docs/reference/kubectl/quick-reference/>
- <https://kubernetes.io/es/docs/reference/>

### Amazon EKS

- <https://docs.aws.amazon.com/eks/latest/userguide/>

### Terraform

- <https://developer.hashicorp.com/terraform>
- <https://developer.hashicorp.com/terraform/cli>
- <https://registry.terraform.io/providers/hashicorp/aws/latest/docs>

### Helm

- <https://helm.sh/docs/>
- <https://helm.sh/docs/helm/>

### GitHub Actions

- <https://docs.github.com/actions>
- <https://docs.github.com/actions/deployment>

---

## Principios operativos recomendados

1. Confirmar identidad y destino antes de modificar infraestructura.
2. Aplicar mínimo privilegio.
3. Preferir OIDC y credenciales temporales frente a claves permanentes.
4. No versionar secretos, kubeconfigs ni states sensibles.
5. Usar `terraform plan`, `kubectl diff`, `helm template` y dry-runs antes de cambios relevantes.
6. Separar CI de CD y proteger producción.
7. Usar state remoto y locking para Terraform colaborativo.
8. Mantener imágenes y releases trazables por commit SHA o versión.
9. Conocer el rollback antes de ejecutar el cambio.
10. Diagnosticar de forma sistemática antes de aplicar correcciones ad hoc.

---

## Licencia y contribuciones

El contenido está orientado a aprendizaje y referencia técnica. Si encontrás un comando desactualizado, una explicación mejorable o un caso práctico que aporte valor, podés abrir un Issue o Pull Request.

---

**Este repositorio no está afiliado oficialmente con Amazon Web Services, HashiCorp, GitHub, Helm ni The Kubernetes Authors/CNCF.**
