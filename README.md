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
$ terraform apply -target="module.karpenter"
$ terraform apply
```

## Architecture

### Metric Architecture

<img src="/images/architecture-metric.png" width="800"/>

### Log Architecture

<img src="/images/architecture-log.png" width="800"/>

### Trace Architecture

<img src="/images/architecture-trace.png" width="800"/>
