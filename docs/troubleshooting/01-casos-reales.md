# Troubleshooting DevOps — Casos reales

Esta sección propone un enfoque sistemático para diagnosticar problemas frecuentes combinando AWS CLI, EKS, kubectl, Helm y Terraform.

La idea no es memorizar errores, sino seguir una secuencia reproducible.

## Método general de diagnóstico

1. Confirmar identidad y destino.
2. Confirmar conectividad y acceso.
3. Revisar estado deseado vs estado real.
4. Revisar eventos y logs.
5. Aislar si el problema es infraestructura, cluster, workload o aplicación.
6. Aplicar el cambio mínimo.
7. Validar recuperación.

---

## Caso 1 — kubectl responde Unauthorized

### Síntoma

```text
You must be logged in to the server (Unauthorized)
```

### Diagnóstico

```bash
aws sts get-caller-identity
kubectl config current-context
kubectl config view --minify
```

Regenerar kubeconfig:

```bash
aws eks update-kubeconfig \
  --region <region> \
  --name <cluster>
```

Validar identidad:

```bash
kubectl auth whoami
```

Si EKS usa access entries:

```bash
aws eks list-access-entries --cluster-name <cluster>
```

### Causas frecuentes

- perfil AWS incorrecto;
- sesión SSO vencida;
- kubeconfig apuntando a otro cluster;
- principal sin acceso al cluster;
- región incorrecta.

---

## Caso 2 — Forbidden aunque kubectl autentica

### Síntoma

```text
Error from server (Forbidden)
```

### Diagnóstico

```bash
kubectl auth whoami
kubectl auth can-i get pods -n <namespace>
kubectl auth can-i create deployments -n <namespace>
```

### Interpretación

Autenticación y autorización son problemas distintos. Si `whoami` funciona pero `can-i` devuelve `no`, el acceso al cluster existe pero faltan permisos RBAC/EKS.

---

## Caso 3 — Pod en CrashLoopBackOff

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

Revisar:

- variables de entorno;
- ConfigMaps y Secrets;
- errores de conexión a bases o APIs;
- probes;
- exit code;
- OOMKilled;
- argumentos de inicio.

Si tiene varios containers:

```bash
kubectl logs <pod> -c <container> -n <namespace>
```

---

## Caso 4 — ImagePullBackOff

```bash
kubectl describe pod <pod> -n <namespace>
```

Ver imagen configurada:

```bash
kubectl get pod <pod> -n <namespace> \
  -o jsonpath='{.spec.containers[*].image}'
```

Para ECR:

```bash
aws ecr describe-repositories
aws ecr list-images --repository-name <repo>
```

Causas frecuentes:

- tag inexistente;
- URI incorrecta;
- permisos insuficientes;
- registry privado sin acceso;
- problemas de red/DNS.

---

## Caso 5 — Pod queda Pending

```bash
kubectl describe pod <pod> -n <namespace>
kubectl get events -n <namespace> --sort-by=.metadata.creationTimestamp
kubectl get nodes
kubectl top nodes
```

Revisar requests:

```bash
kubectl get pod <pod> -n <namespace> \
  -o jsonpath='{.spec.containers[*].resources}'
```

Causas:

- recursos insuficientes;
- nodeSelector incorrecto;
- affinity;
- taints;
- PVC pendiente;
- autoscaling sin capacidad disponible.

---

## Caso 6 — Service no responde

```bash
kubectl get svc -n <namespace>
kubectl describe svc <service> -n <namespace>
kubectl get endpoints -n <namespace>
```

Validar labels y selector:

```bash
kubectl get pods -n <namespace> --show-labels
kubectl get svc <service> -n <namespace> -o yaml
```

Si no hay endpoints, normalmente el selector del Service no coincide con los Pods Ready.

Prueba local:

```bash
kubectl port-forward service/<service> 8080:<port> -n <namespace>
```

---

## Caso 7 — Deployment no completa rollout

```bash
kubectl rollout status deployment/<name> -n <namespace>
kubectl describe deployment <name> -n <namespace>
kubectl get rs -n <namespace>
kubectl get pods -n <namespace>
```

Historial:

```bash
kubectl rollout history deployment/<name> -n <namespace>
```

Rollback:

```bash
kubectl rollout undo deployment/<name> -n <namespace>
```

---

## Caso 8 — Helm upgrade falla

```bash
helm status <release> -n <namespace>
helm history <release> -n <namespace>
helm get values <release> -n <namespace>
helm get manifest <release> -n <namespace>
```

Renderizar antes de reintentar:

```bash
helm lint ./chart
helm template <release> ./chart -f values.yaml
```

Rollback:

```bash
helm rollback <release> <revision> -n <namespace>
```

---

## Caso 9 — Terraform detecta cambios inesperados

Primero:

```bash
terraform plan
```

Inspeccionar state:

```bash
terraform state list
terraform state show <resource>
```

Validar infraestructura real con AWS CLI.

Ejemplo:

```bash
aws ec2 describe-instances
```

Posibles causas:

- drift manual;
- provider actualizado;
- variables diferentes;
- workspace incorrecto;
- state incorrecto;
- cambios en defaults del provider.

No aplicar automáticamente hasta entender por qué existe el diff.

---

## Caso 10 — Terraform state lock

### Síntoma

```text
Error acquiring the state lock
```

Antes de forzar desbloqueo, verificar que no exista otro `terraform apply` activo.

```bash
terraform force-unlock <LOCK_ID>
```

> `force-unlock` sólo debe utilizarse cuando se confirmó que el lock es huérfano.

---

## Caso 11 — AWS CLI AccessDenied

```bash
aws sts get-caller-identity
```

Confirmar perfil:

```bash
aws configure list
```

Probar llamada concreta con debug sólo si hace falta:

```bash
aws <service> <operation> --debug
```

Analizar:

- IAM policies;
- permission boundaries;
- SCPs de AWS Organizations;
- resource policies;
- session policies;
- KMS permissions.

---

## Caso 12 — DNS interno falla dentro de Kubernetes

Crear/debuggear un Pod temporal:

```bash
kubectl run dns-test \
  --image=busybox:1.36 \
  --restart=Never \
  -it --rm \
  -- nslookup kubernetes.default
```

Revisar CoreDNS:

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system -l k8s-app=kube-dns
```

---

## Caso 13 — Aplicación funciona local pero no en cluster

Separar capas:

```text
Aplicación
  ↓
Container
  ↓
Pod
  ↓
Service
  ↓
Ingress / Load Balancer
  ↓
DNS
```

Validar desde adentro hacia afuera:

```bash
kubectl logs <pod>
kubectl exec -it <pod> -- /bin/sh
kubectl port-forward pod/<pod> 8080:<port>
kubectl get svc
kubectl get ingress
```

---

## Caso 14 — Nodo NotReady

```bash
kubectl get nodes
kubectl describe node <node>
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

En EKS además:

```bash
aws eks describe-nodegroup \
  --cluster-name <cluster> \
  --nodegroup-name <nodegroup>
```

Revisar:

- kubelet;
- red/CNI;
- disco;
- memoria;
- IAM;
- estado EC2;
- health del node group.

---

## Caso 15 — Cambio aplicado al cluster equivocado

Prevención:

```bash
aws sts get-caller-identity
kubectl config current-context
kubectl config view --minify
```

Recomendaciones:

- nombres de contexts explícitos;
- namespaces separados;
- prompts que muestren context;
- permisos más estrictos en producción;
- PRs y CI/CD para cambios sensibles.

---

## Checklist de incidente

```bash
aws sts get-caller-identity
kubectl config current-context
kubectl get nodes
kubectl get pods -A
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

Después bajar al recurso específico con `describe`, `logs`, `history` y la herramienta correspondiente.
