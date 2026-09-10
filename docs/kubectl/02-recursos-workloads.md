# 06 — kubectl: recursos y workloads

## Consultar recursos

```bash
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get all
kubectl get all -n mi-namespace
kubectl get pods -A
kubectl get pods -o wide
```

Ver un recurso concreto:

```bash
kubectl get deployment api
kubectl get deployment api -o yaml
```

Descripción detallada:

```bash
kubectl describe pod api-123
kubectl describe deployment api
```

---

## Crear recursos

Desde YAML:

```bash
kubectl apply -f deployment.yaml
```

Directorio completo:

```bash
kubectl apply -f ./k8s/
```

URL remota:

```bash
kubectl apply -f https://ejemplo.com/recurso.yaml
```

Crear imperativamente:

```bash
kubectl create deployment api --image=nginx:alpine
kubectl create service clusterip api --tcp=80:8080
kubectl create namespace desarrollo
```

Generar YAML sin aplicarlo:

```bash
kubectl create deployment api \
  --image=nginx:alpine \
  --dry-run=client \
  -o yaml
```

---

## `apply`, `create`, `replace`

### `apply`

Recomendado para recursos declarativos:

```bash
kubectl apply -f recurso.yaml
```

### `create`

Falla si el recurso ya existe:

```bash
kubectl create -f recurso.yaml
```

### `replace`

Reemplaza el recurso existente:

```bash
kubectl replace -f recurso.yaml
```

Forzar reemplazo puede recrear el recurso y ser disruptivo:

```bash
kubectl replace --force -f recurso.yaml
```

---

## Ver diferencias antes de aplicar

```bash
kubectl diff -f ./k8s/
```

Es una de las mejores defensas contra cambios inesperados.

---

## Editar

```bash
kubectl edit deployment api
```

Para cambios repetibles, preferí editar el manifiesto versionado y aplicar después.

---

## Patch

Merge patch:

```bash
kubectl patch deployment api \
  -p '{"spec":{"replicas":3}}'
```

Strategic merge y JSON patch también están disponibles según recurso y caso de uso.

---

## Eliminar

```bash
kubectl delete pod api-123
kubectl delete deployment api
kubectl delete -f deployment.yaml
```

Por label:

```bash
kubectl delete pods -l app=api
```

Esperar finalización:

```bash
kubectl delete pod api-123 --wait=true
```

> Antes de un delete masivo, ejecutá primero el `get` equivalente con el mismo selector.

---

# Workloads

## Deployment

```bash
kubectl get deployments
kubectl describe deployment api
kubectl create deployment api --image=nginx:alpine
kubectl scale deployment api --replicas=3
kubectl rollout status deployment/api
kubectl rollout history deployment/api
kubectl rollout undo deployment/api
```

Cambiar imagen:

```bash
kubectl set image deployment/api api=mi-registry/api:2.0.0
```

Reiniciar rollout:

```bash
kubectl rollout restart deployment/api
```

---

## ReplicaSet

```bash
kubectl get rs
kubectl describe rs <nombre>
```

Normalmente se administra indirectamente mediante Deployments.

---

## StatefulSet

```bash
kubectl get statefulsets
kubectl get sts
kubectl describe statefulset postgres
kubectl scale statefulset postgres --replicas=3
kubectl rollout status statefulset/postgres
```

---

## DaemonSet

```bash
kubectl get daemonsets
kubectl get ds
kubectl describe daemonset agente
kubectl rollout status daemonset/agente
```

---

## Job

```bash
kubectl get jobs
kubectl create job prueba --image=busybox -- echo hola
kubectl describe job prueba
kubectl logs job/prueba
```

Crear desde CronJob:

```bash
kubectl create job --from=cronjob/backup backup-manual
```

---

## CronJob

```bash
kubectl get cronjobs
kubectl get cj
kubectl describe cronjob backup
```

Suspender:

```bash
kubectl patch cronjob backup -p '{"spec":{"suspend":true}}'
```

---

# Networking

## Service

```bash
kubectl get svc
kubectl describe svc api
kubectl expose deployment api --port=80 --target-port=8080
```

Endpoints relacionados:

```bash
kubectl get endpoints
kubectl get endpointslices
```

## Ingress

```bash
kubectl get ingress
kubectl get ing
kubectl describe ingress api
```

---

# Configuración

## ConfigMap

```bash
kubectl get configmaps
kubectl get cm
kubectl create configmap app-config --from-literal=ENV=production
kubectl create configmap app-config --from-file=.env.example
kubectl describe configmap app-config
```

## Secret

```bash
kubectl get secrets
kubectl create secret generic app-secret \
  --from-literal=API_TOKEN=ejemplo
```

Inspección de metadata:

```bash
kubectl describe secret app-secret
```

> Los Secrets de Kubernetes no deben considerarse automáticamente cifrados de extremo a extremo. La protección depende de la configuración del cluster, etcd, RBAC y del proveedor.

---

# Storage

```bash
kubectl get storageclass
kubectl get sc
kubectl get persistentvolumes
kubectl get pv
kubectl get persistentvolumeclaims
kubectl get pvc
```

Descripción:

```bash
kubectl describe pvc datos
```

---

# Service Accounts y RBAC

```bash
kubectl get serviceaccounts
kubectl get sa
kubectl get roles
kubectl get rolebindings
kubectl get clusterroles
kubectl get clusterrolebindings
```

Comprobar autorización:

```bash
kubectl auth can-i get pods
kubectl auth can-i create deployments -n desarrollo
kubectl auth can-i '*' '*' --all-namespaces
```

Impersonación para diagnóstico, si tenés permiso:

```bash
kubectl auth can-i get pods --as usuario@example.com
```

---

## Generación declarativa útil

```bash
kubectl create namespace desarrollo --dry-run=client -o yaml
kubectl create deployment api --image=nginx --dry-run=client -o yaml
kubectl create service clusterip api --tcp=80:8080 --dry-run=client -o yaml
kubectl create configmap app --from-literal=ENV=dev --dry-run=client -o yaml
```

Esto permite generar una base, revisarla, versionarla y luego aplicar.

---

## Referencias

- Workloads: <https://kubernetes.io/docs/concepts/workloads/>
- Services: <https://kubernetes.io/docs/concepts/services-networking/service/>
- ConfigMaps: <https://kubernetes.io/docs/concepts/configuration/configmap/>
- Secrets: <https://kubernetes.io/docs/concepts/configuration/secret/>
- RBAC: <https://kubernetes.io/docs/reference/access-authn-authz/rbac/>
