# 07 — kubectl: observabilidad, debug y operación

## Logs

Logs de un Pod:

```bash
kubectl logs api-123
```

Seguir en tiempo real:

```bash
kubectl logs -f api-123
```

Contenedor específico:

```bash
kubectl logs api-123 -c api
```

Todos los contenedores:

```bash
kubectl logs api-123 --all-containers=true
```

Instancia anterior del contenedor:

```bash
kubectl logs api-123 --previous
```

Por label:

```bash
kubectl logs -l app=api --all-containers=true
```

Últimos 100 registros:

```bash
kubectl logs api-123 --tail=100
```

Desde hace 30 minutos:

```bash
kubectl logs api-123 --since=30m
```

---

## Ejecutar comandos dentro de un contenedor

```bash
kubectl exec api-123 -- env
kubectl exec api-123 -- ls -la
```

Shell interactiva:

```bash
kubectl exec -it api-123 -- /bin/sh
```

Si existe Bash:

```bash
kubectl exec -it api-123 -- /bin/bash
```

Contenedor específico:

```bash
kubectl exec -it api-123 -c api -- /bin/sh
```

---

## Copiar archivos

```bash
kubectl cp ./archivo.txt default/api-123:/tmp/archivo.txt
kubectl cp default/api-123:/tmp/salida.log ./salida.log
```

`kubectl cp` puede depender de `tar` dentro de la imagen del contenedor.

---

## Port forwarding

Pod:

```bash
kubectl port-forward pod/api-123 8080:8080
```

Deployment:

```bash
kubectl port-forward deployment/api 8080:8080
```

Service:

```bash
kubectl port-forward service/api 8080:80
```

---

## Proxy

```bash
kubectl proxy
```

Permite acceder localmente al API Server mediante un proxy autenticado.

---

## Eventos

```bash
kubectl get events
kubectl get events -A
kubectl get events --sort-by=.metadata.creationTimestamp
```

Eventos de un recurso suelen aparecer también en:

```bash
kubectl describe pod api-123
```

---

## Métricas

Si Metrics Server está disponible:

```bash
kubectl top nodes
kubectl top pods
kubectl top pods -A
kubectl top pod api-123 --containers
```

Si `kubectl top` falla, no significa necesariamente que el cluster esté caído; puede faltar Metrics Server o acceso a métricas.

---

# Rollouts

Estado:

```bash
kubectl rollout status deployment/api
```

Historial:

```bash
kubectl rollout history deployment/api
```

Detalle de revisión:

```bash
kubectl rollout history deployment/api --revision=3
```

Rollback:

```bash
kubectl rollout undo deployment/api
```

A revisión concreta:

```bash
kubectl rollout undo deployment/api --to-revision=2
```

Reinicio:

```bash
kubectl rollout restart deployment/api
```

Pausar / reanudar:

```bash
kubectl rollout pause deployment/api
kubectl rollout resume deployment/api
```

---

## Escalado

Manual:

```bash
kubectl scale deployment api --replicas=5
```

Con precondición:

```bash
kubectl scale deployment api --current-replicas=3 --replicas=5
```

Autoscaling básico:

```bash
kubectl autoscale deployment api --min=2 --max=10 --cpu-percent=70
```

Consultar HPA:

```bash
kubectl get hpa
```

---

# Debug de Pods

## Estado general

```bash
kubectl get pod api-123 -o wide
kubectl describe pod api-123
kubectl logs api-123 --all-containers=true
```

## `Pending`

Revisar:

```bash
kubectl describe pod api-123
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl get nodes
kubectl describe node <node>
kubectl get pvc
```

Causas frecuentes:

- falta de CPU o memoria;
- selector/affinity imposible;
- taints sin tolerations;
- PVC pendiente;
- quota;
- scheduler restrictions.

## `CrashLoopBackOff`

```bash
kubectl describe pod api-123
kubectl logs api-123
kubectl logs api-123 --previous
```

Revisar entrypoint, configuración, secretos, dependencias, probes y errores de la aplicación.

## `ImagePullBackOff` / `ErrImagePull`

```bash
kubectl describe pod api-123
```

Revisar:

- nombre/tag de imagen;
- registry;
- credenciales;
- imagePullSecrets;
- conectividad;
- existencia de la imagen.

## Probes fallando

```bash
kubectl describe pod api-123
kubectl get pod api-123 -o yaml
```

Revisar `startupProbe`, `readinessProbe` y `livenessProbe`.

---

## Debug con contenedores efímeros

Cuando la imagen es mínima y no tiene herramientas:

```bash
kubectl debug -it pod/api-123 --image=busybox
```

Copiar un Pod para debug:

```bash
kubectl debug pod/api-123 -it --copy-to=api-debug --container=debug --image=ubuntu
```

La disponibilidad exacta depende de la versión del cluster y de permisos.

---

# Nodos

Listar:

```bash
kubectl get nodes -o wide
```

Descripción:

```bash
kubectl describe node worker-01
```

## Cordon

Evita nuevos Pods programados en el nodo:

```bash
kubectl cordon worker-01
```

Rehabilitar:

```bash
kubectl uncordon worker-01
```

## Drain

Evacuar workloads antes de mantenimiento:

```bash
kubectl drain worker-01 --ignore-daemonsets
```

Según el cluster puede requerir opciones adicionales, por ejemplo para Pods con `emptyDir`:

```bash
kubectl drain worker-01 --ignore-daemonsets --delete-emptydir-data
```

> `drain` es una operación de mantenimiento potencialmente disruptiva. Revisá PodDisruptionBudgets, réplicas y capacidad del resto del cluster.

---

## Taints

Ver:

```bash
kubectl describe node worker-01
```

Agregar:

```bash
kubectl taint nodes worker-01 dedicated=backend:NoSchedule
```

Eliminar:

```bash
kubectl taint nodes worker-01 dedicated=backend:NoSchedule-
```

---

## Wait

Esperar una condición:

```bash
kubectl wait \
  --for=condition=Ready \
  pod/api-123 \
  --timeout=60s
```

Deployment disponible:

```bash
kubectl wait \
  --for=condition=Available \
  deployment/api \
  --timeout=120s
```

---

## Diagnóstico rápido recomendado

```bash
kubectl config current-context
kubectl get pods -A
kubectl get nodes
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

Después bajar al recurso afectado:

```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --all-containers=true
kubectl logs <pod> -n <namespace> --all-containers=true --previous
```

---

## Referencias

- Debug de aplicaciones: <https://kubernetes.io/docs/tasks/debug/debug-application/>
- Debug de cluster: <https://kubernetes.io/docs/tasks/debug/debug-cluster/>
- Logs: <https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/>
- Drain: <https://kubernetes.io/docs/reference/kubectl/generated/kubectl_drain/>
