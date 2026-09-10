# Helm — Fundamentos, charts y operación profesional

Helm es el gestor de paquetes para Kubernetes. Permite empaquetar manifiestos, parametrizarlos, versionarlos y desplegarlos como releases repetibles.

> La documentación oficial actual incluye Helm 4. La guía mantiene comandos compatibles con el flujo moderno y evita depender de comportamientos obsoletos.

## Conceptos clave

- **Chart**: paquete de recursos Kubernetes parametrizados.
- **Release**: una instalación concreta de un chart en un cluster/namespace.
- **Repository**: fuente de charts.
- **values.yaml**: valores configurables de un chart.
- **templates/**: manifiestos Kubernetes renderizados por Helm.

## Instalación y verificación

```bash
helm version
helm env
```

Documentación oficial:

- <https://helm.sh/docs/intro/install/>
- <https://helm.sh/docs/helm/>
- <https://helm.sh/es/docs/v3/helm/>

## Repositorios

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo list
helm repo update
helm search repo nginx
```

Eliminar un repositorio:

```bash
helm repo remove bitnami
```

## Inspeccionar un chart antes de instalar

```bash
helm show chart <repo/chart>
helm show values <repo/chart>
helm show readme <repo/chart>
```

Renderizar localmente:

```bash
helm template mi-release <repo/chart>
```

Con valores propios:

```bash
helm template mi-release <repo/chart> -f values.yaml
```

Validar un chart propio:

```bash
helm lint ./mi-chart
```

## Instalar

```bash
helm install mi-release <repo/chart>
```

En namespace específico:

```bash
helm install mi-release <repo/chart> \
  --namespace mi-app \
  --create-namespace
```

Con valores:

```bash
helm install mi-release <repo/chart> \
  -f values.yaml
```

Overrides puntuales:

```bash
helm install mi-release <repo/chart> \
  --set replicaCount=3
```

Antes de instalar:

```bash
helm install mi-release <repo/chart> \
  --dry-run \
  --debug
```

## Releases instaladas

```bash
helm list
helm list -A
helm status mi-release
helm history mi-release
```

Ver valores efectivos:

```bash
helm get values mi-release
helm get values mi-release --all
```

Ver manifiestos aplicados:

```bash
helm get manifest mi-release
```

## Upgrade

```bash
helm upgrade mi-release <repo/chart>
```

Patrón idempotente muy usado en CI/CD:

```bash
helm upgrade --install mi-release <repo/chart> \
  --namespace mi-app \
  --create-namespace \
  -f values.yaml
```

Con espera y timeout:

```bash
helm upgrade --install mi-release ./chart \
  --namespace mi-app \
  --wait \
  --timeout 5m
```

## Rollback

Ver historial:

```bash
helm history mi-release
```

Volver a una revisión:

```bash
helm rollback mi-release 2
```

Con limpieza si falla:

```bash
helm rollback mi-release 2 --cleanup-on-fail
```

## Uninstall

```bash
helm uninstall mi-release
```

En namespace específico:

```bash
helm uninstall mi-release -n mi-app
```

## Crear un chart

```bash
helm create mi-api
```

Estructura típica:

```text
mi-api/
├── Chart.yaml
├── values.yaml
├── charts/
└── templates/
```

## Dependencias

```bash
helm dependency list ./mi-chart
helm dependency update ./mi-chart
helm dependency build ./mi-chart
```

## Empaquetado

```bash
helm package ./mi-chart
```

## OCI registries

Helm puede trabajar con registries OCI:

```bash
helm registry login <registry>
helm push mi-chart-1.0.0.tgz oci://<registry>/<ruta>
```

## Buenas prácticas

1. Versionar `Chart.yaml`, `values.yaml` y templates.
2. Separar valores por ambiente cuando corresponda.
3. No guardar secretos reales en `values.yaml` versionados.
4. Ejecutar `helm lint` y `helm template` en CI.
5. Usar `helm upgrade --install` para despliegues repetibles.
6. Usar `--wait` y timeout explícito en automatizaciones.
7. Revisar `helm history` antes de rollback.
8. Evitar `--force` salvo que se entienda el impacto de reemplazar recursos.

## Secuencia operativa recomendada

```bash
helm lint ./chart
helm template mi-app ./chart -f values-prod.yaml
helm upgrade --install mi-app ./chart \
  -n produccion \
  --create-namespace \
  -f values-prod.yaml \
  --wait \
  --timeout 5m
helm status mi-app -n produccion
```

## Documentación oficial

- <https://helm.sh/docs/>
- <https://helm.sh/docs/helm/>
- <https://helm.sh/docs/intro/cheatsheet/>
