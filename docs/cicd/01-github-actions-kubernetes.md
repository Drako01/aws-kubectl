# CI/CD con Kubernetes — GitHub Actions, AWS ECR, EKS y Helm

Este capítulo muestra un pipeline de referencia para construir una imagen, publicarla en Amazon ECR y desplegarla en Amazon EKS con `kubectl` o Helm.

El objetivo es entender el flujo. Los nombres, permisos, regiones, cuentas, branches y políticas deben adaptarse a cada proyecto.

## Flujo general

```text
Push / Pull Request
       │
       ▼
GitHub Actions
       │
       ├── test / lint / build
       │
       ├── build Docker image
       │
       ├── authenticate to AWS via OIDC
       │
       ├── push image to ECR
       │
       ├── update kubeconfig for EKS
       │
       └── deploy with kubectl or Helm
```

## Por qué OIDC

Para GitHub Actions, preferir OIDC evita guardar access keys AWS de larga duración como secrets del repositorio.

El workflow obtiene credenciales temporales asumiendo un IAM Role autorizado.

Documentación oficial:

- <https://docs.github.com/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services>
- <https://github.com/aws-actions/configure-aws-credentials>

## Permisos mínimos del workflow

```yaml
permissions:
  contents: read
  id-token: write
```

`id-token: write` permite solicitar el token OIDC. No otorga por sí mismo permisos sobre AWS.

## Variables recomendadas

```yaml
env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: mi-api
  EKS_CLUSTER: mi-cluster
  K8S_NAMESPACE: produccion
```

## Pipeline con kubectl

Ejemplo conceptual:

```yaml
name: Deploy to EKS

on:
  push:
    branches: [main]

permissions:
  contents: read
  id-token: write

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: mi-api
  EKS_CLUSTER: mi-cluster
  K8S_NAMESPACE: produccion

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v5
        with:
          role-to-assume: ${{ secrets.AWS_DEPLOY_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and push image
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t "$REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" .
          docker push "$REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG"

      - name: Configure kubectl
        run: |
          aws eks update-kubeconfig \
            --region "$AWS_REGION" \
            --name "$EKS_CLUSTER"

      - name: Deploy
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          kubectl set image \
            deployment/mi-api \
            api="$REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" \
            -n "$K8S_NAMESPACE"

          kubectl rollout status \
            deployment/mi-api \
            -n "$K8S_NAMESPACE" \
            --timeout=5m
```

## Pipeline con Helm

Si la aplicación usa un chart:

```yaml
- name: Deploy with Helm
  env:
    REGISTRY: ${{ steps.login-ecr.outputs.registry }}
    IMAGE_TAG: ${{ github.sha }}
  run: |
    helm upgrade --install mi-api ./helm/mi-api \
      --namespace "$K8S_NAMESPACE" \
      --create-namespace \
      --set image.repository="$REGISTRY/$ECR_REPOSITORY" \
      --set image.tag="$IMAGE_TAG" \
      --wait \
      --timeout 5m
```

## Validaciones previas al deploy

Antes de desplegar conviene ejecutar:

```yaml
- name: Validate Helm chart
  run: |
    helm lint ./helm/mi-api
    helm template mi-api ./helm/mi-api > /tmp/rendered.yaml
```

Para manifiestos puros:

```yaml
- name: Validate Kubernetes manifests
  run: |
    kubectl apply --dry-run=client -f ./k8s/
```

## Separar CI de CD

Un diseño profesional suele separar:

```text
Pull Request
  └── lint
  └── tests
  └── build
  └── terraform validate/plan
  └── helm lint/template

Merge a main
  └── build image
  └── push ECR
  └── deploy staging/production
```

Esto evita desplegar cambios que todavía no pasaron revisión.

## Ambientes

GitHub Environments permiten separar, por ejemplo:

```text
development
staging
production
```

Y asociar:

- secrets;
- variables;
- reglas de protección;
- aprobaciones requeridas.

Para producción, una aprobación manual puede ser una buena barrera adicional.

## Tags de imagen

Evitar depender exclusivamente de `latest`.

Una opción robusta:

```text
<git-sha>
```

Ejemplo:

```bash
IMAGE_TAG=${GITHUB_SHA}
```

Esto mejora trazabilidad y rollback.

## Rollback con kubectl

```bash
kubectl rollout history deployment/mi-api -n produccion
kubectl rollout undo deployment/mi-api -n produccion
```

## Rollback con Helm

```bash
helm history mi-api -n produccion
helm rollback mi-api <revision> -n produccion
```

## Terraform dentro de CI

Para IaC:

```yaml
- name: Terraform validate
  run: |
    terraform fmt -check -recursive
    terraform init -input=false
    terraform validate
```

Para PR:

```yaml
- name: Terraform plan
  run: terraform plan -input=false
```

`apply` debería quedar restringido a un flujo explícito, revisado y con credenciales/roles específicos.

## Seguridad

1. Preferir OIDC frente a access keys permanentes.
2. Usar IAM Roles con mínimo privilegio.
3. Limitar `role-to-assume` al repo/branch/environment correcto mediante trust policy.
4. No imprimir secretos ni tokens.
5. Fijar versiones mayores de Actions conocidas y mantenerlas actualizadas.
6. Separar roles de CI, staging y producción.
7. Proteger el environment `production`.
8. Evitar `kubectl apply` indiscriminado sobre directorios no revisados.
9. Esperar explícitamente el rollout y fallar el pipeline si no termina.
10. Conservar trazabilidad entre commit SHA, imagen y release.

## Pipeline recomendado

```text
CI
├── lint
├── test
├── build
├── terraform fmt/validate/plan
├── helm lint/template
└── security checks

CD
├── AWS OIDC
├── Docker build
├── ECR push
├── EKS kubeconfig
├── Helm upgrade --install
├── rollout/status
└── smoke test
```

## Documentación oficial

- <https://docs.github.com/actions>
- <https://docs.github.com/actions/deployment>
- <https://docs.aws.amazon.com/eks/latest/userguide/>
- <https://helm.sh/docs/>
- <https://kubernetes.io/docs/concepts/workloads/controllers/deployment/>
