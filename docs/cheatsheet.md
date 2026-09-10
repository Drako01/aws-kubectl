# Cheat Sheet — AWS CLI + kubectl

## AWS CLI

### Identidad y configuración

```bash
aws --version
aws configure list
aws configure list-profiles
aws sts get-caller-identity
aws configure get region
```

### Ayuda

```bash
aws help
aws ec2 help
aws ec2 describe-instances help
```

### S3

```bash
aws s3 ls
aws s3 ls s3://<bucket>/
aws s3 cp archivo.txt s3://<bucket>/
aws s3 sync ./dist s3://<bucket>/
```

### EC2

```bash
aws ec2 describe-instances
aws ec2 describe-vpcs
aws ec2 describe-subnets
aws ec2 describe-security-groups
```

### ECR / ECS / EKS

```bash
aws ecr describe-repositories
aws ecs list-clusters
aws ecs list-services --cluster <cluster>
aws eks list-clusters
aws eks describe-cluster --name <cluster>
aws eks update-kubeconfig --region <region> --name <cluster>
```

### Lambda / Logs / SSM

```bash
aws lambda list-functions
aws lambda get-function --function-name <function>
aws logs describe-log-groups
aws logs tail <log-group> --follow
aws ssm describe-instance-information
```

### RDS / DynamoDB / CloudFormation

```bash
aws rds describe-db-instances
aws dynamodb list-tables
aws dynamodb describe-table --table-name <table>
aws cloudformation describe-stacks --stack-name <stack>
```

### Opciones globales

```text
--profile <perfil>
--region <region>
--output json|table|text|yaml
--query '<JMESPath>'
--no-cli-pager
```

---

## kubectl

### Context y cluster

```bash
kubectl version --client
kubectl cluster-info
kubectl config get-contexts
kubectl config current-context
kubectl config use-context <context>
kubectl config view --minify
```

### Recursos

```bash
kubectl api-resources
kubectl get pods
kubectl get pods -A
kubectl get pods -o wide
kubectl get deployments
kubectl get services
kubectl get nodes
```

### Detalle y manifiestos

```bash
kubectl describe pod <pod>
kubectl explain deployment.spec
kubectl diff -f ./k8s/
kubectl apply -f ./k8s/
```

### Namespace

```bash
kubectl get ns
kubectl get pods -n <namespace>
kubectl config set-context --current --namespace=<namespace>
```

### Logs y acceso

```bash
kubectl logs <pod>
kubectl logs -f <pod>
kubectl logs <pod> --previous
kubectl exec -it <pod> -- /bin/sh
kubectl port-forward service/<service> 8080:80
```

### Rollout y escala

```bash
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout restart deployment/<name>
kubectl rollout undo deployment/<name>
kubectl scale deployment <name> --replicas=3
```

### Labels, eventos y métricas

```bash
kubectl get pods --show-labels
kubectl get pods -l app=api
kubectl get events -A --sort-by=.metadata.creationTimestamp
kubectl top nodes
kubectl top pods -A
```

### Autorización y nodos

```bash
kubectl auth can-i get pods
kubectl auth whoami
kubectl cordon <node>
kubectl uncordon <node>
```

### Wait y debug

```bash
kubectl wait --for=condition=Ready pod/<pod> --timeout=60s
kubectl debug -it pod/<pod> --image=busybox
```

---

## Validación previa a cambios importantes

```bash
aws sts get-caller-identity
kubectl config current-context
kubectl config view --minify
```
