# 08 — kubectl: referencia de comandos

Esta página funciona como índice explicado. Para flags exhaustivos de cada comando utilizá:

```bash
kubectl <comando> --help
```

y la referencia oficial: <https://kubernetes.io/docs/reference/kubectl/generated/>.

## Información y descubrimiento

| Comando | Uso |
| --- | --- |
| `kubectl version` | Muestra versiones de cliente y servidor |
| `kubectl cluster-info` | Información básica del cluster |
| `kubectl api-resources` | Lista tipos de recursos disponibles |
| `kubectl api-versions` | Lista versiones de API disponibles |
| `kubectl explain` | Explica esquema y campos de recursos |
| `kubectl options` | Lista opciones globales |

## Lectura

| Comando | Uso |
| --- | --- |
| `kubectl get` | Lista o consulta recursos |
| `kubectl describe` | Muestra detalle y eventos asociados |
| `kubectl logs` | Lee logs de contenedores |
| `kubectl top` | Métricas de CPU/memoria si Metrics API está disponible |
| `kubectl events` | Lista eventos con interfaz orientada a eventos |

## Creación y gestión declarativa

| Comando | Uso |
| --- | --- |
| `kubectl create` | Crea recursos |
| `kubectl apply` | Aplica configuración declarativa |
| `kubectl diff` | Compara estado actual con manifiestos |
| `kubectl replace` | Reemplaza recursos existentes |
| `kubectl delete` | Elimina recursos |

## Modificación

| Comando | Uso |
| --- | --- |
| `kubectl edit` | Edita un recurso en el editor |
| `kubectl patch` | Modifica campos mediante patch |
| `kubectl label` | Administra labels |
| `kubectl annotate` | Administra annotations |
| `kubectl set` | Modifica aspectos concretos de recursos |
| `kubectl scale` | Cambia cantidad de réplicas |
| `kubectl autoscale` | Crea/configura autoscaling horizontal básico |

## Deployments y rollouts

| Comando | Uso |
| --- | --- |
| `kubectl rollout status` | Espera/consulta estado del rollout |
| `kubectl rollout history` | Muestra historial |
| `kubectl rollout undo` | Revierte |
| `kubectl rollout restart` | Reinicia rollout |
| `kubectl rollout pause` | Pausa |
| `kubectl rollout resume` | Reanuda |

## Ejecución y acceso

| Comando | Uso |
| --- | --- |
| `kubectl exec` | Ejecuta un comando en un contenedor |
| `kubectl attach` | Adjunta stdin/stdout a un proceso existente |
| `kubectl cp` | Copia archivos entre local y contenedor |
| `kubectl port-forward` | Reenvía puertos locales hacia Pods/Services |
| `kubectl proxy` | Proxy local hacia el API Server |
| `kubectl run` | Crea/ejecuta un Pod según opciones |
| `kubectl debug` | Crea recursos o contenedores de diagnóstico |

## Nodos y mantenimiento

| Comando | Uso |
| --- | --- |
| `kubectl cordon` | Marca un nodo como no schedulable |
| `kubectl uncordon` | Habilita scheduling nuevamente |
| `kubectl drain` | Evacúa workloads para mantenimiento |
| `kubectl taint` | Administra taints de nodos |

## Configuración de acceso

| Comando | Uso |
| --- | --- |
| `kubectl config view` | Muestra kubeconfig |
| `kubectl config get-contexts` | Lista contexts |
| `kubectl config current-context` | Context actual |
| `kubectl config use-context` | Cambia context |
| `kubectl config set-context` | Crea/modifica context |
| `kubectl config set-cluster` | Configura cluster |
| `kubectl config set-credentials` | Configura credenciales |
| `kubectl config unset` | Elimina una propiedad |
| `kubectl config delete-context` | Elimina context |
| `kubectl config rename-context` | Renombra context |

## Seguridad y autorización

| Comando | Uso |
| --- | --- |
| `kubectl auth can-i` | Comprueba si una acción está autorizada |
| `kubectl auth reconcile` | Reconcilia reglas RBAC desde archivos |
| `kubectl auth whoami` | Muestra atributos de identidad cuando el servidor lo soporta |
| `kubectl create role` | Crea Role |
| `kubectl create rolebinding` | Crea RoleBinding |
| `kubectl create clusterrole` | Crea ClusterRole |
| `kubectl create clusterrolebinding` | Crea ClusterRoleBinding |
| `kubectl create serviceaccount` | Crea ServiceAccount |

## Espera y condiciones

| Comando | Uso |
| --- | --- |
| `kubectl wait` | Espera una condición de uno o más recursos |

## Plugins y comandos externos

| Comando | Uso |
| --- | --- |
| `kubectl plugin list` | Lista plugins ejecutables detectados |

Los plugins siguen el patrón de ejecutables `kubectl-<nombre>` presentes en `PATH`.

## `kubectl create`: subcomandos frecuentes

```text
clusterrole
clusterrolebinding
configmap
cronjob
deployment
ingress
job
namespace
poddisruptionbudget
priorityclass
quota
role
rolebinding
secret
service
serviceaccount
token
```

Consultar:

```bash
kubectl create --help
```

## `kubectl set`: subcomandos frecuentes

```text
env
image
resources
selector
serviceaccount
subject
```

Ejemplos:

```bash
kubectl set image deployment/api api=repo/api:2.0
kubectl set env deployment/api LOG_LEVEL=debug
kubectl set resources deployment/api -c api --limits=cpu=500m,memory=512Mi
```

## Flags globales importantes

```text
--context
--namespace / -n
--kubeconfig
--cluster
--user
--server
--request-timeout
--as
--as-group
--v
```

Ejemplo explícito y seguro:

```bash
kubectl \
  --context produccion \
  --namespace backend \
  get pods
```

## Shortnames frecuentes

```text
po     pods
deploy deployments
rs     replicasets
sts    statefulsets
ds     daemonsets
svc    services
ing    ingresses
cm     configmaps
ns     namespaces
no     nodes
pv     persistentvolumes
pvc    persistentvolumeclaims
sa     serviceaccounts
cj     cronjobs
```

No memorices shortnames dudosos: consultalos en tu cluster con:

```bash
kubectl api-resources
```

## Referencia oficial completa

- Comandos generados: <https://kubernetes.io/docs/reference/kubectl/generated/>
- Quick Reference: <https://kubernetes.io/docs/reference/kubectl/quick-reference/>
- Convenciones: <https://kubernetes.io/docs/reference/kubectl/conventions/>
