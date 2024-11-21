# eks-adot Project

eks-adot is prototyping project for ADOT(AWS Distro for OpenTelemetry) collector on EKS. eks-adot project consists of the following git repositories.

* [aws-terraform](https://github.com/ssup2-playground/eks-adot_aws-terraform) : Terraform for EKS cluster and ADOT collectors.
* [app-python](https://github.com/ssup2-playground/eks-adot_app-python) : Sample python application.

## Install

* Run terraform

```shell
# Get terraform code
$ git clone https://github.com/ssup2-playground/eks-adot_aws-terraform.git && rm ./eks-adot_aws-terraform/terraform.tf

# Run terraform
$ cd eks-adot_aws-terraform
$ terraform init
$ terraform apply -target="module.prometheus"
$ terraform apply -target="awscc_osis_pipeline.metrics"
$ terraform apply -target="awscc_osis_pipeline.logs"
$ terraform apply -target="awscc_osis_pipeline.traces"
$ terraform apply -target="module.irsa_observer_adot_metric_cw"
$ terraform apply -target="module.irsa_observer_adot_metric_amp"
$ terraform apply -target="module.irsa_observer_adot_metric_os"
$ terraform apply -target="module.irsa_observer_adot_log_cw"
$ terraform apply -target="module.irsa_observer_adot_log_os"
$ terraform apply -target="module.irsa_observer_adot_trace_xray"
$ terraform apply -target="module.irsa_observer_adot_trace_os"
$ terraform apply -target="module.karpenter"
$ terraform apply
```

* Set Loki, Tempo endpoints to ADOT collectors in work EKS cluster.

```shell
# Get Loki and Tempo endpoints on ob EKS cluster
$ aws eks update-kubeconfig --name eks-adot-ob-eks
$ LOKI_ENDPOINT=$(kubectl -n adot-collector get service adot-logd-loki --output jsonpath='{.status.loadBalancer.ingress[0].hostname}')
$ TEMPO_ENDPOINT=$(kubectl -n adot-collector get service adot-traced-tempo --output jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

* Run terraform again for ADOT collectors in work EKS cluster.

```shell
# Run terraform with endpoint vars
$ terraform apply -var "loki-logd-endpoint=$LOKI_ENDPOINT" -var "tempo-traced-endpoint=$TEMPO_ENDPOINT"
```

* Restart workloads for auto instrumentations.

```shell
# restart workloads in observer EKS cluster
$ aws eks update-kubeconfig --name eks-adot-ob-eks
$ kubectl -n app rollout restart deployment app-python-xray
$ kubectl -n app rollout restart deployment app-python-os
$ kubectl -n app rollout restart deployment app-python-tempo

# restart workloads in work EKS cluster
$ aws eks update-kubeconfig --name eks-adot-work-eks
$ kubectl -n app rollout restart deployment app-python
```

## Login Grafana

* Set grafana NLB

```bash
$ aws eks update-kubeconfig --name eks-adot-ob-eks
$ kubectl -n observability patch service grafana -p '{"spec": {"type": "LoadBalancer"}}'
```

* Set grafana NLB security Group

```shell
$ MY_IP=$(curl -s https://checkip.amazonaws.com/)
$ SG_ID=$(aws ec2 describe-security-groups --filters Name=tag:Name,Values=eks-adot-grafana-sg --query "SecurityGroups[*].GroupId" --output text)
$ aws ec2 authorize-security-group-ingress --group-id "$SG_ID" --protocol tcp --port 80 --cidr "$MY_IP/32"
```

* Get grafana `admin` user password

```shell
$ kubectl -n observability get secrets grafana -o jsonpath='{.data.admin-password}' | base64 --decode
ugnQJC5Sgg3WkuHi7k8le4U3oB1f9EKhj2G4uS48
```

* Get grafana endpoint

```shell
$ echo http://$(kubectl -n observability get service grafana --output jsonpath='{.status.loadBalancer.ingress[0].hostname}')
http://k8s-observab-grafana-e4e76fb41d-a64e31df8c64616d.elb.ap-northeast-2.amazonaws.com
```

* Login grafana

<img src="/images/grafana-login.png" width="400"/>

## Login and Set Permission OpenSearch

* Get my IP address

```shell
$ curl -s https://checkip.amazonaws.com/
```

* Set my IP address to opensearch security config

<img src="/images/opensearch-login-01.png" width="600"/>

<img src="/images/opensearch-login-02.png" width="800"/>

<img src="/images/opensearch-login-03.png" width="800"/>

* Get OpenSearch Dashboard URL

<img src="/images/opensearch-login-04.png" width="800"/>

* Login with ID `admin` / Password `Admin123!`

<img src="/images/opensearch-login-05.png" width="600"/>

<img src="/images/opensearch-login-06.png" width="600"/>

* Get ARN for IAM Role ARN Ingest

```bash
$ aws iam get-role --role-name eks-adot-opensearch-injest --query Role.Arn
```

* Set ARN for IAM Role ARN Ingest

<img src="/images/opensearch-permission-01.png" width="800"/>

<img src="/images/opensearch-permission-02.png" width="800"/>

<img src="/images/opensearch-permission-03.png" width="800"/>

<img src="/images/opensearch-permission-04.png" width="800"/>

<img src="/images/opensearch-permission-05.png" width="800"/>

<img src="/images/opensearch-permission-06.png" width="800"/>

## Architecture

### Metric Architecture

<img src="/images/architecture-metric.png" width="800"/>

### Log Architecture

<img src="/images/architecture-log.png" width="800"/>

### Trace Architecture

<img src="/images/architecture-trace.png" width="800"/>

## Guide & Screenshot

### Metric / CloudWatch

* Screenshots

<img src="/images/metric-cloudwatch-01.png" width="800"/>

### Metric / AMP

* Screenshots

<img src="/images/metric-amp-01.png" width="800"/>

### Metric / OpenSearch

* Set index patterns

<img src="/images/metric-opensearch-index-pattern-01.png" width="800"/>

<img src="/images/metric-opensearch-index-pattern-02.png" width="800"/>

<img src="/images/metric-opensearch-index-pattern-03.png" width="800"/>

<img src="/images/metric-opensearch-index-pattern-04.png" width="800"/>

<img src="/images/metric-opensearch-index-pattern-05.png" width="800"/>

* Screenshots

<img src="/images/metric-opensearch-discover-01.png" width="800"/>

### Log / CloudWatch

* Screenshots

<img src="/images/log-cloudwatch-01.png" width="800"/>

<img src="/images/log-cloudwatch-02.png" width="800"/>

### Log / OpenSearch

* Set index patterns

<img src="/images/log-opensearch-index-pattern-01.png" width="800"/>

<img src="/images/log-opensearch-index-pattern-02.png" width="800"/>

<img src="/images/log-opensearch-index-pattern-03.png" width="800"/>

<img src="/images/log-opensearch-index-pattern-04.png" width="800"/>

<img src="/images/log-opensearch-index-pattern-05.png" width="800"/>

* Screenshots

<img src="/images/log-opensearch-discover-01.png" width="800"/>

### Log / Loki

* Screenshots

<img src="/images/log-loki-01.png" width="800"/>

### Log / Loki Work

* Screenshots

<img src="/images/log-loki-work-01.png" width="800"/>

### Trace / X-Ray

* Screenshots

<img src="/images/trace-xray-01.png" width="800"/>

<img src="/images/trace-xray-02.png" width="800"/>

### Trace / OpenSearch

* Screenshots

<img src="/images/trace-opensearch-01.png" width="800"/>

<img src="/images/trace-opensearch-02.png" width="800"/>

### Trace / Tempo

* Screenshots

<img src="/images/trace-tempo-01.png" width="800"/>

### Trace / Tempo Work

* Screenshots

<img src="/images/trace-tempo-work-01.png" width="800"/>