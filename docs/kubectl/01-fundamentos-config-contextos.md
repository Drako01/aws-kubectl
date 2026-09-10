# 05 — kubectl: fundamentos, kubeconfig y contexts

## Qué es kubectl

`kubectl` es el cliente de línea de comandos de Kubernetes. Se comunica con el API Server usando la configuración almacenada normalmente en un archivo `kubeconfig`.

Sintaxis general:

```bash
kubectl <comando> <tipo> [nombre] [flags]
```

Ejemplos:

```bash
kubectl get pods
kubectl describe deployment api
kubectl apply -f deployment.yaml
```

---

## Verificar instalación y versión

```bash
kubectl version --client
```

Información del cliente y servidor:

```bash
kubectl version
```

Documentación oficial:

<https://kubernetes.io/docs/tasks/tools/>

---

## Ayuda integrada

```bash
kubectl help
kubectl get --help
kubectl create --help
kubectl create deployment --help
```

API resources disponibles en el cluster:

```bash
kubectl api-resources
```

Versiones de API:

```bash
kubectl api-versions
```

Explicar un recurso:

```bash
kubectl explain pod
kubectl explain deployment
kubectl explain deployment.spec
kubectl explain deployment.spec.template.spec.containers
```

`kubectl explain` es especialmente útil al escribir YAML sin salir de la terminal.

---

## Kubeconfig

Ubicación habitual:

```text
~/.kube/config
```

Windows:

```text
%USERPROFILE%\.kube\config
```

Usar otro archivo:

```bash
kubectl --kubeconfig ./config get pods
```

Variable de entorno:

```bash
export KUBECONFIG=~/.kube/config
```

PowerShell:

```powershell
$env:KUBECONFIG="$HOME\.kube\config"
```

Nunca publiques kubeconfigs con credenciales o tokens.

---

## Contexts

Un context combina típicamente:

- cluster;
- usuario/credencial;
- namespace por defecto.

Listar:

```bash
kubectl config get-contexts
```

Context actual:

```bash
kubectl config current-context
```

Cambiar:

```bash
kubectl config use-context desarrollo
```

Ver configuración:

```bash
kubectl config view
```

Ver configuración efectiva minimizada:

```bash
kubectl config view --minify
```

Renombrar context:

```bash
kubectl config rename-context contexto-viejo contexto-nuevo
```

Eliminar context:

```bash
kubectl config delete-context contexto-viejo
```

---

## Regla operativa fundamental

Antes de una modificación importante:

```bash
kubectl config current-context
kubectl config view --minify
```

Y si el namespace importa:

```bash
kubectl config view --minify --output 'jsonpath={..namespace}'
```

En Kubernetes, equivocarse de context puede ser más peligroso que equivocarse de comando.

---

## Namespaces

Listar:

```bash
kubectl get namespaces
```

Forma corta:

```bash
kubectl get ns
```

Consultar un namespace:

```bash
kubectl get pods -n desarrollo
```

Todos:

```bash
kubectl get pods -A
```

Definir namespace por defecto en el context actual:

```bash
kubectl config set-context --current --namespace=desarrollo
```

Crear:

```bash
kubectl create namespace desarrollo
```

Eliminar:

```bash
kubectl delete namespace desarrollo
```

> Eliminar un namespace elimina sus recursos namespaced. Verificá el impacto antes de ejecutar el comando.

---

## Cluster info

```bash
kubectl cluster-info
```

Información de nodos:

```bash
kubectl get nodes
kubectl get nodes -o wide
```

Versión:

```bash
kubectl version
```

---

## Formatos de salida

```bash
kubectl get pods -o wide
kubectl get pods -o yaml
kubectl get pods -o json
kubectl get pods -o name
```

JSONPath:

```bash
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
```

Custom columns:

```bash
kubectl get pods \
  -o custom-columns='NAME:.metadata.name,STATUS:.status.phase,NODE:.spec.nodeName'
```

---

## Labels

Mostrar:

```bash
kubectl get pods --show-labels
```

Filtrar:

```bash
kubectl get pods -l app=api
kubectl get pods -l 'environment in (dev,staging)'
```

Agregar/modificar label:

```bash
kubectl label pod api-123 environment=dev
```

Sobrescribir:

```bash
kubectl label pod api-123 environment=staging --overwrite
```

Eliminar label:

```bash
kubectl label pod api-123 environment-
```

---

## Annotations

```bash
kubectl annotate deployment api owner=backend
kubectl annotate deployment api owner=platform --overwrite
kubectl annotate deployment api owner-
```

Labels se usan principalmente para selección y organización; annotations para metadata no destinada normalmente a selección.

---

## Autocomplete

Bash:

```bash
source <(kubectl completion bash)
```

Alias frecuente:

```bash
alias k=kubectl
```

Para que el alias tenga completion en Bash:

```bash
complete -o default -F __start_kubectl k
```

Zsh:

```bash
source <(kubectl completion zsh)
```

---

## Recursos más comunes

```text
pods / po
deployments / deploy
replicasets / rs
statefulsets / sts
daemonsets / ds
services / svc
ingresses / ing
configmaps / cm
secrets
jobs
cronjobs / cj
namespaces / ns
nodes / no
persistentvolumes / pv
persistentvolumeclaims / pvc
serviceaccounts / sa
roles
rolebindings
clusterroles
clusterrolebindings
```

Los shortnames disponibles dependen del cluster y pueden consultarse con:

```bash
kubectl api-resources
```

---

## Recursos oficiales

- kubectl: <https://kubernetes.io/docs/reference/kubectl/>
- Quick Reference: <https://kubernetes.io/docs/reference/kubectl/quick-reference/>
- kubeconfig: <https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/>
- instalación: <https://kubernetes.io/docs/tasks/tools/>
