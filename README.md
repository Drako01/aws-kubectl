# DevOps Toolkit — Guía Profesional

<p align="center">
  <a href="https://aws.amazon.com/cli/" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/amazonwebservices/amazonwebservices-original-wordmark.svg" alt="Amazon Web Services" width="115" />
  </a>
  &nbsp;&nbsp;&nbsp;
  <a href="https://kubernetes.io/docs/reference/kubectl/" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/kubernetes/kubernetes-plain-wordmark.svg" alt="Kubernetes" width="95" />
  </a>
  &nbsp;&nbsp;&nbsp;
  <a href="https://developer.hashicorp.com/terraform" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/terraform/terraform-original-wordmark.svg" alt="Terraform" width="105" />
  </a>
  &nbsp;&nbsp;&nbsp;
  <a href="https://helm.sh/docs/" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/helm/helm-original.svg" alt="Helm" width="54" />
  </a>
  &nbsp;&nbsp;&nbsp;
  <a href="https://docs.github.com/actions" target="_blank" rel="noreferrer">
    <img src="https://raw.githubusercontent.com/devicons/devicon/master/icons/githubactions/githubactions-original.svg" alt="GitHub Actions" width="54" />
  </a>
</p>

<p align="center">
  <strong>Guía práctica en español para aprender, consultar y operar herramientas DevOps modernas.</strong>
</p>

<p align="center">
  <a href="https://aws.amazon.com/cli/"><img src="https://img.shields.io/badge/AWS%20CLI-v2-FF9900?logo=amazonaws&logoColor=white" alt="AWS CLI v2" /></a>
  <a href="https://kubernetes.io/docs/reference/kubectl/"><img src="https://img.shields.io/badge/kubectl-Kubernetes-326CE5?logo=kubernetes&logoColor=white" alt="kubectl" /></a>
  <a href="https://developer.hashicorp.com/terraform"><img src="https://img.shields.io/badge/Terraform-IaC-844FBA?logo=terraform&logoColor=white" alt="Terraform" /></a>
  <a href="https://helm.sh/docs/"><img src="https://img.shields.io/badge/Helm-Kubernetes-0F1689?logo=helm&logoColor=white" alt="Helm" /></a>
  <a href="https://docs.github.com/actions"><img src="https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-2088FF?logo=githubactions&logoColor=white" alt="GitHub Actions" /></a>
  <img src="https://img.shields.io/badge/Idioma-Espa%C3%B1ol-informational" alt="Español" />
</p>

<p align="center">
  <strong>Autor:</strong> <a href="https://github.com/Drako01">Alejandro Di Stefano</a>
</p>

---

## ¿Qué es este repositorio?

**DevOps Toolkit** es una guía de aprendizaje y referencia operativa organizada por herramientas y escenarios reales.

Está pensada para consultar rápidamente cómo trabajar con infraestructura Cloud, Kubernetes, Infrastructure as Code, empaquetado de aplicaciones, automatización y troubleshooting sin tener que recorrer documentación dispersa.

Cada sección explica conceptos, comandos, riesgos, buenas prácticas y flujos de trabajo reales. No reemplaza la documentación oficial: funciona como una capa práctica y ordenada sobre ella.

---

# Navegación rápida

| Área | Para qué sirve | Ir a |
| --- | --- | --- |
| ☁️ **AWS CLI** | Administrar servicios AWS desde terminal | [Ver guía de AWS CLI](#aws-cli) |
| ☸️ **Kubernetes / kubectl** | Operar clusters, workloads y recursos Kubernetes | [Ver guía de kubectl](#kubernetes--kubectl) |
| 🏗️ **Terraform** | Gestionar infraestructura como código | [Ver guía de Terraform](#terraform) |
| ⎈ **Helm** | Gestionar charts y releases Kubernetes | [Ver guía de Helm](#helm) |
| 🔶 **Amazon EKS** | Operar Kubernetes administrado en AWS | [Ver guía de EKS](#amazon-eks) |
| ⚙️ **CI/CD** | Automatizar build, publicación y despliegues | [Ver CI/CD](#cicd-con-github-actions) |
| 🧯 **Troubleshooting** | Diagnosticar problemas reales paso a paso | [Ver troubleshooting](#troubleshooting) |
| ⚡ **Cheat Sheet** | Consulta rápida de comandos frecuentes | [Abrir Cheat Sheet](docs/cheatsheet.md) |

---

# AWS CLI

Automatización y operación de servicios de Amazon Web Services desde línea de comandos.

### Contenido

1. [Fundamentos, instalación y configuración](docs/aws/01-fundamentos-instalacion-configuracion.md)  
   Instalación, estructura de comandos, regiones, configuración y ayuda integrada.

2. [Credenciales, perfiles, outputs y consultas](docs/aws/02-credenciales-perfiles-output.md)  
   IAM, SSO, profiles, variables, JMESPath, JSON, table y text.

3. [Servicios esenciales](docs/aws/03-servicios-esenciales.md)  
   STS, IAM, S3, EC2, ECR, ECS, EKS, Lambda, CloudWatch, SSM, RDS, DynamoDB, Route 53, CloudFormation y más.

4. [Automatización, seguridad y troubleshooting](docs/aws/04-automatizacion-seguridad-troubleshooting.md)  
   Bash, PowerShell, paginación, retries, seguridad, dry-run y diagnóstico.

**Documentación oficial:** [AWS CLI](https://docs.aws.amazon.com/cli/)

---

# Kubernetes / kubectl

Operación, administración y diagnóstico de clusters Kubernetes mediante `kubectl`.

### Contenido

1. [Fundamentos, kubeconfig y contexts](docs/kubectl/01-fundamentos-config-contextos.md)  
   Clusters, contexts, namespaces y configuración local.

2. [Recursos y workloads](docs/kubectl/02-recursos-workloads.md)  
   Pods, Deployments, StatefulSets, DaemonSets, Jobs, Services, ConfigMaps, Secrets y storage.

3. [Observabilidad, debugging y operación](docs/kubectl/03-observabilidad-debug-operacion.md)  
   Logs, exec, port-forward, events, metrics, rollout, scale, drain, cordon y nodos.

4. [Referencia de comandos](docs/kubectl/04-referencia-comandos.md)  
   Índice explicado de los principales comandos de `kubectl`.

**Documentación oficial:** [kubectl](https://kubernetes.io/docs/reference/kubectl/)

---

# Terraform

Infrastructure as Code para definir infraestructura reproducible, versionable y auditable.

### Contenido

1. [Fundamentos, instalación y workflow](docs/terraform/01-fundamentos-instalacion-workflow.md)  
   HCL, providers, `init`, `fmt`, `validate`, `plan`, `apply`, variables y outputs.

2. [State, backends e importación](docs/terraform/02-state-backends-import.md)  
   State local/remoto, locking, backends, import, drift, moved blocks y workspaces.

3. [AWS, módulos, seguridad y troubleshooting](docs/terraform/03-aws-modulos-seguridad-troubleshooting.md)  
   AWS Provider, módulos, `for_each`, `count`, lifecycle, secretos, EKS y diagnóstico.

**Documentación oficial:** [Terraform](https://developer.hashicorp.com/terraform)

---

# Helm

Gestión de aplicaciones Kubernetes mediante charts, values y releases versionadas.

### Contenido

- [Helm: charts y operación profesional](docs/helm/01-fundamentos-charts-operacion.md)

Incluye repositories, charts, `values.yaml`, `helm lint`, `helm template`, instalación, upgrades, rollbacks, releases, OCI y buenas prácticas.

**Documentación oficial:** [Helm](https://helm.sh/docs/)

---

# Amazon EKS

Kubernetes administrado en AWS, combinando AWS CLI, IAM, ECR, EKS y `kubectl`.

### Contenido

1. [EKS con AWS CLI y kubectl](docs/eks/01-aws-eks-kubectl.md)  
   Kubeconfig, acceso, autenticación, ECR y flujo operativo básico.

2. [Operación avanzada de EKS](docs/eks/02-operacion-avanzada.md)  
   Access Entries, Managed Node Groups, add-ons, upgrades, Pod Identity y troubleshooting.

**Documentación oficial:** [Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/)

---

# CI/CD con GitHub Actions

Automatización de pipelines para aplicaciones desplegadas sobre AWS y Kubernetes.

### Contenido

- [GitHub Actions + Kubernetes](docs/cicd/01-github-actions-kubernetes.md)

Incluye:

- CI de tests, lint y build;
- autenticación en AWS mediante OIDC;
- build de imágenes Docker;
- publicación en Amazon ECR;
- despliegues en EKS;
- `kubectl` y Helm;
- environments;
- separación CI/CD;
- rollback;
- trazabilidad por commit SHA.

**Documentación oficial:** [GitHub Actions](https://docs.github.com/actions)

---

# Troubleshooting

Problemas reales de operación explicados como secuencias de diagnóstico, no como recetas aisladas.

### Casos incluidos

- `Unauthorized` y `Forbidden`;
- `CrashLoopBackOff`;
- `ImagePullBackOff`;
- Pods en `Pending`;
- Nodes `NotReady`;
- errores DNS/CoreDNS;
- Services sin endpoints;
- rollouts fallidos;
- errores de Helm;
- Terraform drift y locks de state;
- AWS `AccessDenied`;
- problemas de acceso a EKS;
- fallos relacionados con imágenes y ECR.

➡️ [Abrir casos reales de troubleshooting](docs/troubleshooting/01-casos-reales.md)

---

# Cheat Sheet

Para consulta rápida durante el trabajo diario:

➡️ **[AWS CLI + kubectl + Terraform + Helm — Cheat Sheet](docs/cheatsheet.md)**

---

## Cómo se relacionan las herramientas

```text
GitHub
  │
  ▼
GitHub Actions
  │
  ├── CI: test / lint / build
  ├── Terraform plan/apply
  └── Docker build + push
              │
              ▼
             AWS
        ┌─────┴─────┐
        ▼           ▼
       ECR         EKS
                    │
              ┌─────┴─────┐
              ▼           ▼
           kubectl       Helm
              │           │
              └─────┬─────┘
                    ▼
             Kubernetes
```

---

## Ruta sugerida de aprendizaje

Si estás empezando, un recorrido lógico sería:

```text
1. AWS CLI
2. Kubernetes / kubectl
3. Amazon EKS
4. Terraform
5. Helm
6. CI/CD con GitHub Actions
7. Troubleshooting
```

No es obligatorio seguir este orden. Cada bloque puede utilizarse como referencia independiente.

---

## Principios de la guía

- comandos acompañados de contexto;
- foco en operación real;
- seguridad y mínimo privilegio;
- infraestructura reproducible;
- cambios revisables antes de producción;
- credenciales temporales y OIDC cuando corresponde;
- separación clara entre infraestructura y workloads;
- rollback conocido antes de aplicar cambios críticos;
- troubleshooting sistemático antes de ejecutar correcciones ad hoc.

---

## Referencias oficiales

| Tecnología | Documentación |
| --- | --- |
| AWS CLI | https://docs.aws.amazon.com/cli/ |
| Kubernetes / kubectl | https://kubernetes.io/docs/reference/kubectl/ |
| Amazon EKS | https://docs.aws.amazon.com/eks/latest/userguide/ |
| Terraform | https://developer.hashicorp.com/terraform |
| Terraform AWS Provider | https://registry.terraform.io/providers/hashicorp/aws/latest/docs |
| Helm | https://helm.sh/docs/ |
| GitHub Actions | https://docs.github.com/actions |

---

## Contribuciones

Si encontrás un comando desactualizado, una explicación mejorable o un escenario que debería incorporarse, podés abrir un Issue o Pull Request.

Ver [CONTRIBUTING.md](CONTRIBUTING.md).

---

**DevOps Toolkit** no está afiliado oficialmente con Amazon Web Services, HashiCorp, GitHub, Helm ni The Kubernetes Authors/CNCF.
