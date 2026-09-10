# Contribuir

Gracias por ayudar a mejorar esta guía.

## Alcance

Este repositorio acepta mejoras relacionadas con:

- AWS CLI v2;
- kubectl y Kubernetes;
- Amazon EKS;
- ejemplos reproducibles;
- troubleshooting;
- correcciones de documentación;
- actualización de enlaces oficiales.

## Criterios

Las contribuciones deberían:

1. estar escritas en español claro;
2. explicar qué hace el comando y cuándo usarlo;
3. evitar credenciales, tokens o datos reales;
4. priorizar comandos actuales;
5. enlazar documentación oficial cuando agreguen conceptos sensibles a versión;
6. advertir cuando una operación pueda ser disruptiva;
7. mantener ejemplos genéricos y reproducibles.

## Flujo sugerido

```bash
git switch main
git pull --ff-only
git switch -c docs/nombre-del-cambio
```

Luego:

```bash
git add .
git commit -m "docs: describe cambio"
git push -u origin docs/nombre-del-cambio
```

Abrí un Pull Request explicando brevemente el objetivo, alcance y fuentes oficiales utilizadas.

## Estilo

- Markdown simple y legible.
- Títulos jerárquicos.
- Bloques de código con lenguaje (`bash`, `powershell`, `yaml`, etc.).
- Variables reemplazables usando `<valor>`.
- No asumir que el lector conoce el contexto previo.

## Fuentes

Priorizamos documentación de primera parte:

- <https://docs.aws.amazon.com/cli/>
- <https://kubernetes.io/docs/>
- <https://docs.aws.amazon.com/eks/>
