# eks-adot Project

eks-adot is prototyping project for ADOT(AWS Distro for OpenTelemetry) collector on EKS. eks-adot project consists of the following git repositories.

* [aws-terraform](https://github.com/ssup2-playground/eks-adot_aws-terraform) : Terraform for EKS cluster and ADOT collectors.
* [app-python](https://github.com/ssup2-playground/eks-adot_app-python) : Sample python application.

## Install

* Run terraform

```bash
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

* Restart workloads for auto instrumentations.

```
# restart workloads in observer EKS cluster
$ aws eks update-kubeconfig --name eks-adot-ob-eks
$ kubectl -n app rollout restart deployment app-python-xray
$ kubectl -n app rollout restart deployment app-python-os
$ kubectl -n app rollout restart deployment app-python-tempo

# restart workloads in work EKS cluster
$ aws eks update-kubeconfig --name eks-adot-work-eks
$ kubectl -n app rollout restart deployment app-python
```

* Set Loki, Tempo endpoints to ADOT collectors in work EKS cluster

```
# Get Loki and Tempo endpoints on ob EKS cluster
$ aws eks update-kubeconfig --name eks-adot-ob-eks
$ 

```

## Login Grafana

* Set grafana NLB
```
$ kubectl -n observability patch service grafana -p '{"spec": {"type": "LoadBalancer"}}'
```

* Set grafana NLB security Group
```
$ MY_IP=$(curl -s https://checkip.amazonaws.com/)
$ SG_ID=$(aws ec2 describe-security-groups --filters Name=tag:Name,Values=eks-adot-grafana-sg --query "SecurityGroups[*].GroupId" --output text)
$ aws ec2 authorize-security-group-ingress --group-id "$SG_ID" --protocol tcp --port 80 --cidr "$MY_IP/32"
```

* Get grafana `admin` user password
```
$ kubectl -n observability get secrets grafana -o jsonpath='{.data.admin-password}' | base64 --decode
ugnQJC5Sgg3WkuHi7k8le4U3oB1f9EKhj2G4uS48
```

* Get grafana endpoint
```shell
$ echo http://$(kubectl -n observability get service grafana --output jsonpath='{.status.loadBalancer.ingress[0].hostname}')
http://k8s-observab-grafana-e4e76fb41d-a64e31df8c64616d.elb.ap-northeast-2.amazonaws.com
```

## Login OpenSearch

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

### Log / OpenSearch

### Log / Loki

### Log / Loki Work

### Trace / X-Ray

### Trace / OpenSearch

### Trace / Tempo

### Trace / Tempo Work
