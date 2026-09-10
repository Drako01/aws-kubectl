# 03 — AWS CLI: servicios esenciales

Esta sección reúne operaciones frecuentes de los servicios más usados. Para cobertura exhaustiva de cualquier servicio consultá la referencia oficial: <https://docs.aws.amazon.com/cli/latest/reference/>.

## STS

Identidad actual:

```bash
aws sts get-caller-identity
```

Assume role:

```bash
aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/ReadOnly \
  --role-session-name cli-session
```

## IAM

```bash
aws iam list-users
aws iam list-roles
aws iam get-user
aws iam get-role --role-name MiRol
aws iam list-attached-user-policies --user-name usuario
aws iam list-attached-role-policies --role-name MiRol
aws iam list-policies --scope Local
```

Crear usuario:

```bash
aws iam create-user --user-name ejemplo
```

> Evitá access keys permanentes si podés resolver el acceso con roles o SSO.

## S3

Comandos de alto nivel:

```bash
aws s3 ls
aws s3 ls s3://mi-bucket/
aws s3 cp archivo.txt s3://mi-bucket/
aws s3 cp s3://mi-bucket/archivo.txt .
aws s3 sync ./dist s3://mi-bucket/
aws s3 mv archivo.txt s3://mi-bucket/
aws s3 rm s3://mi-bucket/archivo.txt
```

Recursivo:

```bash
aws s3 cp ./carpeta s3://mi-bucket/carpeta --recursive
aws s3 rm s3://mi-bucket/prefijo --recursive
```

API de bajo nivel:

```bash
aws s3api list-buckets
aws s3api get-bucket-versioning --bucket mi-bucket
aws s3api get-bucket-encryption --bucket mi-bucket
aws s3api get-public-access-block --bucket mi-bucket
```

## EC2

```bash
aws ec2 describe-instances
aws ec2 describe-images --owners self
aws ec2 describe-security-groups
aws ec2 describe-vpcs
aws ec2 describe-subnets
aws ec2 describe-volumes
```

Filtrar instancias running:

```bash
aws ec2 describe-instances \
  --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[].Instances[].{Id:InstanceId,Type:InstanceType,IP:PrivateIpAddress}' \
  --output table
```

Estado:

```bash
aws ec2 start-instances --instance-ids i-xxxxxxxx
aws ec2 stop-instances --instance-ids i-xxxxxxxx
aws ec2 reboot-instances --instance-ids i-xxxxxxxx
aws ec2 terminate-instances --instance-ids i-xxxxxxxx
```

## ECR

```bash
aws ecr describe-repositories
aws ecr list-images --repository-name mi-app
aws ecr describe-images --repository-name mi-app
```

Login Docker:

```bash
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.us-east-1.amazonaws.com
```

## ECS

```bash
aws ecs list-clusters
aws ecs list-services --cluster mi-cluster
aws ecs list-tasks --cluster mi-cluster
aws ecs describe-services --cluster mi-cluster --services api
aws ecs describe-tasks --cluster mi-cluster --tasks <task-arn>
```

Forzar nuevo deployment:

```bash
aws ecs update-service \
  --cluster mi-cluster \
  --service api \
  --force-new-deployment
```

## EKS

```bash
aws eks list-clusters
aws eks describe-cluster --name mi-cluster
aws eks list-nodegroups --cluster-name mi-cluster
aws eks describe-nodegroup --cluster-name mi-cluster --nodegroup-name workers
```

Actualizar kubeconfig:

```bash
aws eks update-kubeconfig --region us-east-1 --name mi-cluster
```

Ver el capítulo específico de EKS para el flujo completo.

## Lambda

```bash
aws lambda list-functions
aws lambda get-function --function-name mi-funcion
aws lambda get-function-configuration --function-name mi-funcion
aws lambda list-versions-by-function --function-name mi-funcion
```

Invocar:

```bash
aws lambda invoke \
  --function-name mi-funcion \
  --payload '{"ping":true}' \
  response.json
```

Actualizar código ZIP:

```bash
aws lambda update-function-code \
  --function-name mi-funcion \
  --zip-file fileb://function.zip
```

## CloudWatch Logs

```bash
aws logs describe-log-groups
aws logs describe-log-streams --log-group-name /aws/lambda/mi-funcion
```

Tail en tiempo real:

```bash
aws logs tail /aws/lambda/mi-funcion --follow
```

Última hora:

```bash
aws logs tail /aws/lambda/mi-funcion --since 1h
```

## CloudWatch Metrics / Alarms

```bash
aws cloudwatch describe-alarms
aws cloudwatch list-metrics --namespace AWS/EC2
```

## Systems Manager (SSM)

Instancias administradas:

```bash
aws ssm describe-instance-information
```

Session Manager:

```bash
aws ssm start-session --target i-xxxxxxxx
```

Parameter Store:

```bash
aws ssm get-parameter --name /app/config
aws ssm get-parameter --name /app/secret --with-decryption
aws ssm put-parameter --name /app/config --type String --value ejemplo --overwrite
```

## Secrets Manager

```bash
aws secretsmanager list-secrets
aws secretsmanager describe-secret --secret-id mi-secreto
aws secretsmanager get-secret-value --secret-id mi-secreto
```

No imprimas secretos en logs de CI/CD.

## RDS

```bash
aws rds describe-db-instances
aws rds describe-db-clusters
aws rds describe-db-snapshots
```

Crear snapshot:

```bash
aws rds create-db-snapshot \
  --db-instance-identifier mi-db \
  --db-snapshot-identifier mi-db-manual-001
```

## DynamoDB

```bash
aws dynamodb list-tables
aws dynamodb describe-table --table-name Usuarios
aws dynamodb scan --table-name Usuarios
aws dynamodb query --table-name Usuarios --key-condition-expression 'pk = :pk' --expression-attribute-values '{":pk":{"S":"USER#1"}}'
```

> En tablas grandes evitá usar `scan` como patrón de acceso normal.

## Route 53

```bash
aws route53 list-hosted-zones
aws route53 list-resource-record-sets --hosted-zone-id ZXXXXXXXX
```

## CloudFormation

```bash
aws cloudformation list-stacks
aws cloudformation describe-stacks --stack-name mi-stack
aws cloudformation describe-stack-events --stack-name mi-stack
```

Validar template:

```bash
aws cloudformation validate-template --template-body file://template.yaml
```

Deploy:

```bash
aws cloudformation deploy \
  --template-file template.yaml \
  --stack-name mi-stack \
  --capabilities CAPABILITY_NAMED_IAM
```

## SQS

```bash
aws sqs list-queues
aws sqs get-queue-url --queue-name mi-cola
aws sqs get-queue-attributes --queue-url <url> --attribute-names All
```

## SNS

```bash
aws sns list-topics
aws sns list-subscriptions
aws sns publish --topic-arn <arn> --message "Prueba"
```

## API Gateway

REST API:

```bash
aws apigateway get-rest-apis
```

HTTP/WebSocket API:

```bash
aws apigatewayv2 get-apis
```

## EventBridge

```bash
aws events list-rules
aws events list-targets-by-rule --rule mi-regla
```

## Auto Scaling

```bash
aws autoscaling describe-auto-scaling-groups
aws autoscaling describe-auto-scaling-instances
```

## El patrón para cualquier servicio

Cuando no recuerdes una operación:

```bash
aws <servicio> help
```

Y luego:

```bash
aws <servicio> <operacion> help
```

Ejemplo:

```bash
aws cloudfront help
aws cloudfront list-distributions help
```

La referencia completa y actual de todos los servicios está en:

<https://docs.aws.amazon.com/cli/latest/reference/>
