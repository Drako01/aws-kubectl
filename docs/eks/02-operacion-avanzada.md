# Amazon EKS — Operación avanzada

Este capítulo profundiza en operación diaria y diagnóstico de Amazon EKS usando AWS CLI, kubectl y herramientas asociadas.

## Ver clusters y estado

```bash
aws eks list-clusters
aws eks describe-cluster --name <cluster>
```

Campos relevantes:

```bash
aws eks describe-cluster \
  --name <cluster> \
  --query 'cluster.{status:status,version:version,endpoint:endpoint,role:roleArn}' \
  --output table
```

## Actualizar kubeconfig

```bash
aws eks update-kubeconfig \
  --region <region> \
  --name <cluster>
```

Usar perfil concreto:

```bash
aws eks update-kubeconfig \
  --region <region> \
  --name <cluster> \
  --profile <profile>
```

Validar:

```bash
kubectl config current-context
kubectl cluster-info
kubectl get nodes
```

## Access Entries y autorización

EKS moderno permite administrar acceso al cluster mediante access entries y access policies.

Listar entries:

```bash
aws eks list-access-entries --cluster-name <cluster>
```

Describir una entry:

```bash
aws eks describe-access-entry \
  --cluster-name <cluster> \
  --principal-arn <arn>
```

Listar políticas asociadas:

```bash
aws eks list-associated-access-policies \
  --cluster-name <cluster> \
  --principal-arn <arn>
```

Desde Kubernetes, validar permisos efectivos:

```bash
kubectl auth whoami
kubectl auth can-i get pods -A
kubectl auth can-i create deployments -n <namespace>
```

## Managed Node Groups

Listar:

```bash
aws eks list-nodegroups --cluster-name <cluster>
```

Describir:

```bash
aws eks describe-nodegroup \
  --cluster-name <cluster> \
  --nodegroup-name <nodegroup>
```

Ver scaling config:

```bash
aws eks describe-nodegroup \
  --cluster-name <cluster> \
  --nodegroup-name <nodegroup> \
  --query 'nodegroup.scalingConfig'
```

Actualizar escalado:

```bash
aws eks update-nodegroup-config \
  --cluster-name <cluster> \
  --nodegroup-name <nodegroup> \
  --scaling-config minSize=2,maxSize=6,desiredSize=3
```

## Add-ons

Listar:

```bash
aws eks list-addons --cluster-name <cluster>
```

Describir:

```bash
aws eks describe-addon \
  --cluster-name <cluster> \
  --addon-name <addon>
```

Ver versiones disponibles:

```bash
aws eks describe-addon-versions \
  --addon-name <addon> \
  --kubernetes-version <version>
```

## Versiones y upgrades

Ver versión del control plane:

```bash
aws eks describe-cluster \
  --name <cluster> \
  --query 'cluster.version' \
  --output text
```

Ver versión de nodos:

```bash
kubectl get nodes -o wide
```

Antes de un upgrade:

```bash
kubectl get pods -A
kubectl get pdb -A
kubectl get nodes
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

La estrategia recomendable es validar primero compatibilidad de workloads, add-ons, APIs y node groups.

## EKS Pod Identity

Cuando se usa Pod Identity, conviene inspeccionar asociaciones:

```bash
aws eks list-pod-identity-associations \
  --cluster-name <cluster>
```

Describir una asociación:

```bash
aws eks describe-pod-identity-association \
  --cluster-name <cluster> \
  --association-id <id>
```

## ECR y autenticación Docker

```bash
aws ecr get-login-password --region <region> \
  | docker login \
      --username AWS \
      --password-stdin <account>.dkr.ecr.<region>.amazonaws.com
```

Listar repositorios:

```bash
aws ecr describe-repositories
```

Listar imágenes:

```bash
aws ecr list-images --repository-name <repo>
```

## Diagnóstico de ImagePullBackOff

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
```

Revisar:

- nombre y tag de la imagen;
- existencia en ECR;
- permisos del node role o identidad utilizada;
- conectividad hacia ECR;
- eventos del Pod.

## Diagnóstico de Pending Pods

```bash
kubectl describe pod <pod> -n <namespace>
kubectl get nodes
kubectl top nodes
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
```

Causas frecuentes:

- CPU/memoria insuficiente;
- taints sin tolerations;
- affinity/anti-affinity imposible de satisfacer;
- PVC sin bind;
- límites de red/IP;
- node selectors incompatibles.

## Taints y scheduling

```bash
kubectl describe node <node>
kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
```

Agregar taint:

```bash
kubectl taint nodes <node> workload=critical:NoSchedule
```

Eliminarlo:

```bash
kubectl taint nodes <node> workload=critical:NoSchedule-
```

## Cordon y drain

```bash
kubectl cordon <node>
kubectl drain <node> \
  --ignore-daemonsets \
  --delete-emptydir-data
kubectl uncordon <node>
```

Antes de drenar, revisar PodDisruptionBudgets:

```bash
kubectl get pdb -A
```

## Observabilidad operativa

```bash
kubectl top nodes
kubectl top pods -A
kubectl get events -A --sort-by=.metadata.creationTimestamp
kubectl get pods -A -o wide
```

Para investigar un deployment:

```bash
kubectl rollout status deployment/<name> -n <namespace>
kubectl rollout history deployment/<name> -n <namespace>
kubectl describe deployment <name> -n <namespace>
```

## Checklist antes de operar producción

```bash
aws sts get-caller-identity
aws configure get region
kubectl config current-context
kubectl config view --minify
kubectl auth whoami
```

Confirmar además:

- cuenta AWS;
- región;
- cluster;
- namespace;
- identidad activa;
- permisos efectivos;
- impacto del cambio;
- rollback disponible.

## Documentación oficial

- <https://docs.aws.amazon.com/eks/latest/userguide/>
- <https://docs.aws.amazon.com/eks/latest/userguide/access-entries.html>
- <https://docs.aws.amazon.com/eks/latest/userguide/pod-identities.html>
- <https://docs.aws.amazon.com/eks/latest/userguide/eks-add-ons.html>
