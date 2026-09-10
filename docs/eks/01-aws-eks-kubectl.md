# 09 — Amazon EKS con AWS CLI y kubectl

Amazon EKS es el punto donde AWS CLI y `kubectl` se integran de forma directa: AWS CLI descubre y configura el acceso al cluster; `kubectl` opera los recursos Kubernetes dentro de ese cluster.

## Prerrequisitos

Comprobar herramientas:

```bash
aws --version
kubectl version --client
```

Validar identidad AWS:

```bash
aws sts get-caller-identity
```

Listar clusters:

```bash
aws eks list-clusters --region us-east-1
```

---

## Describir un cluster

```bash
aws eks describe-cluster \
  --name mi-cluster \
  --region us-east-1
```

Resumen útil:

```bash
aws eks describe-cluster \
  --name mi-cluster \
  --region us-east-1 \
  --query 'cluster.{Name:name,Status:status,Version:version,Endpoint:endpoint}' \
  --output table
```

---

## Crear o actualizar kubeconfig

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name mi-cluster
```

Con perfil:

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name mi-cluster \
  --profile desarrollo
```

Con alias de context más legible:

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name mi-cluster \
  --alias empresa-desarrollo
```

Luego:

```bash
kubectl config get-contexts
kubectl config current-context
kubectl cluster-info
kubectl get nodes
```

---

## No asumir que `update-kubeconfig` otorga permisos

`aws eks update-kubeconfig` configura cómo llegar y autenticarse contra el cluster. No garantiza por sí solo que la identidad AWS tenga autorización para todas las operaciones Kubernetes.

Si esto funciona:

```bash
aws sts get-caller-identity
```

pero esto falla:

```bash
kubectl get pods
```

revisá autorización de acceso a EKS y RBAC, no solamente credenciales locales.

---

## Validar acceso

```bash
kubectl auth can-i get pods -A
kubectl auth can-i create deployments -n desarrollo
```

Ver identidad Kubernetes si el cluster/versión lo soporta:

```bash
kubectl auth whoami
```

---

## Node groups

Listar:

```bash
aws eks list-nodegroups --cluster-name mi-cluster
```

Describir:

```bash
aws eks describe-nodegroup \
  --cluster-name mi-cluster \
  --nodegroup-name workers
```

Desde Kubernetes:

```bash
kubectl get nodes -o wide
kubectl describe node <nombre>
```

---

## Add-ons de EKS

```bash
aws eks list-addons --cluster-name mi-cluster
```

Describir:

```bash
aws eks describe-addon \
  --cluster-name mi-cluster \
  --addon-name coredns
```

Versiones compatibles:

```bash
aws eks describe-addon-versions --addon-name coredns
```

---

# Flujo con ECR

## Obtener cuenta

```bash
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
```

## Login

```bash
aws ecr get-login-password --region us-east-1 | \
  docker login \
  --username AWS \
  --password-stdin \
  "$AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com"
```

## Build

```bash
docker build -t mi-api:1.0.0 .
```

## Tag

```bash
docker tag mi-api:1.0.0 \
  "$AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/mi-api:1.0.0"
```

## Push

```bash
docker push \
  "$AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/mi-api:1.0.0"
```

## Actualizar Deployment

```bash
kubectl set image deployment/mi-api \
  mi-api="$AWS_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/mi-api:1.0.0"
```

Seguir rollout:

```bash
kubectl rollout status deployment/mi-api
```

Rollback si corresponde:

```bash
kubectl rollout undo deployment/mi-api
```

---

# Diagnóstico EKS

## 1. ¿AWS funciona?

```bash
aws sts get-caller-identity
aws eks describe-cluster --name mi-cluster --region us-east-1
```

Si falla aquí, el problema está antes de Kubernetes: perfil, región, IAM, SSO, red o nombre de cluster.

## 2. ¿El kubeconfig apunta al cluster correcto?

```bash
kubectl config current-context
kubectl config view --minify
```

## 3. ¿El API Server responde?

```bash
kubectl cluster-info
```

## 4. ¿Hay autorización?

```bash
kubectl auth can-i get pods -A
```

## 5. ¿Los nodos están sanos?

```bash
kubectl get nodes
kubectl describe node <node>
```

## 6. ¿Los Pods están sanos?

```bash
kubectl get pods -A
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

## 7. ¿Hay problemas de aplicación?

```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --all-containers=true
kubectl logs <pod> -n <namespace> --previous --all-containers=true
```

---

## Error: `You must be logged in to the server`

Revisar:

```bash
aws sts get-caller-identity
aws eks update-kubeconfig --region <region> --name <cluster>
kubectl config current-context
```

Si utilizás SSO:

```bash
aws sso login --profile <perfil>
```

---

## Error: `Unauthorized` o `Forbidden`

Diferencia conceptual:

- **autenticación**: quién sos;
- **autorización**: qué podés hacer.

Comprobar identidad AWS:

```bash
aws sts get-caller-identity
```

Comprobar permiso Kubernetes:

```bash
kubectl auth can-i get pods -A
```

Revisar Access Entries / políticas de acceso EKS y RBAC según el modelo configurado en el cluster.

---

## Contextos claros para múltiples clusters

Usar alias reduce errores:

```bash
aws eks update-kubeconfig \
  --name app-dev \
  --region us-east-1 \
  --alias empresa-dev

aws eks update-kubeconfig \
  --name app-prod \
  --region us-east-1 \
  --alias empresa-prod
```

Luego:

```bash
kubectl config get-contexts
kubectl config use-context empresa-dev
```

Antes de producción:

```bash
kubectl config current-context
aws sts get-caller-identity
```

---

## Flujo operativo recomendado

```bash
# 1. Identidad AWS
aws sts get-caller-identity --profile produccion

# 2. Kubeconfig
aws eks update-kubeconfig \
  --profile produccion \
  --region us-east-1 \
  --name app-prod \
  --alias app-prod

# 3. Context
kubectl config current-context

# 4. Salud
kubectl get nodes
kubectl get pods -A

# 5. Diferencia declarativa
kubectl diff -f ./k8s/

# 6. Aplicar
kubectl apply -f ./k8s/

# 7. Verificar
kubectl rollout status deployment/api -n produccion
```

---

## Referencias oficiales

- EKS + kubeconfig: <https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html>
- AWS CLI EKS: <https://docs.aws.amazon.com/cli/latest/reference/eks/>
- Acceso a clusters EKS: <https://docs.aws.amazon.com/eks/latest/userguide/cluster-auth.html>
- ECR: <https://docs.aws.amazon.com/cli/latest/reference/ecr/>
- kubectl: <https://kubernetes.io/docs/reference/kubectl/>
