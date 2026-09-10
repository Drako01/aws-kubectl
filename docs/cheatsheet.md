# Cheat Sheet — DevOps Toolkit

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
aws ecr list-images --repository-name <repo>
aws ecs list-clusters
aws ecs list-services --cluster <cluster>
aws eks list-clusters
aws eks describe-cluster --name <cluster>
aws eks update-kubeconfig --region <region> --name <cluster>
aws eks list-access-entries --cluster-name <cluster>
aws eks list-nodegroups --cluster-name <cluster>
aws eks list-addons --cluster-name <cluster>
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
kubectl apply --dry-run=client -f ./k8s/
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
kubectl drain <node> --ignore-daemonsets
kubectl uncordon <node>
```

### Wait y debug

```bash
kubectl wait --for=condition=Ready pod/<pod> --timeout=60s
kubectl debug -it pod/<pod> --image=busybox
```

---

## Terraform

### Workflow básico

```bash
terraform version
terraform init
terraform fmt -check -recursive
terraform validate
terraform plan
terraform apply
```

### Plan guardado

```bash
terraform plan -out=tfplan
terraform show tfplan
terraform apply tfplan
```

### Variables y outputs

```bash
terraform plan -var-file="prod.tfvars"
terraform output
terraform output -json
```

### State

```bash
terraform state list
terraform state show <address>
terraform state mv <origen> <destino>
terraform state rm <address>
```

### Import

```bash
terraform import <address> <id>
```

### Workspaces

```bash
terraform workspace list
terraform workspace show
terraform workspace new dev
terraform workspace select dev
```

### Providers y diagnóstico

```bash
terraform providers
terraform show
terraform console
TF_LOG=DEBUG terraform plan
```

### Destrucción

```bash
terraform plan -destroy
terraform destroy
```

---

## Helm

### Versión y entorno

```bash
helm version
helm env
```

### Repositorios

```bash
helm repo add <nombre> <url>
helm repo list
helm repo update
helm search repo <termino>
```

### Inspección y validación

```bash
helm show chart <repo/chart>
helm show values <repo/chart>
helm lint ./chart
helm template mi-app ./chart -f values.yaml
```

### Install / Upgrade

```bash
helm install mi-app ./chart
helm upgrade --install mi-app ./chart \
  --namespace mi-app \
  --create-namespace \
  --wait \
  --timeout 5m
```

### Releases

```bash
helm list -A
helm status mi-app -n <namespace>
helm history mi-app -n <namespace>
helm get values mi-app -n <namespace>
```

### Rollback

```bash
helm rollback mi-app <revision> -n <namespace>
```

### Uninstall

```bash
helm uninstall mi-app -n <namespace>
```

---

## Validación previa a cambios importantes

```bash
aws sts get-caller-identity
aws configure get region
kubectl config current-context
kubectl config view --minify
kubectl auth whoami
terraform workspace show
terraform plan
helm list -A
```

## Diagnóstico rápido Kubernetes

```bash
kubectl get nodes
kubectl get pods -A
kubectl get events -A --sort-by=.metadata.creationTimestamp
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```
