# GenAIInfra

GenAIInfra is the containerization and cloud native suite for OPEA, including artifacts to deploy [GenAIExamples](https://github.com/opea-project/GenAIExamples) in a cloud native way, which can be used by enterprise users to deploy to their own cloud.

## Overview

The GenAIInfra repository is organized under four main directories, which include artifacts for OPEA deploying:

| Directory                 | Purpose                                                                                                                                                                                                                                                                                                                                                                            |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `authN-authZ`             | Authentication and Authorization scenarios for OPEA.                                                                                                                                                                                                                                                                                                                               |
| `helm-charts`             | Helm charts for deploying [GenAIComponents](https://github.com/opea-project/GenAIComps) on Kubernetes.                                                                                                                                                                                                                                                                             |
| `kubeai`                  | KubeAI operator for OPEA inference microservices (OIM) deployment, autoscaling and load balancing. More details in [README](kubeai/README.md)                                                                                                                                                                                                                                      |
| `microservices-connector` | GenAI Microservices Connector (GMC) supports the launching, monitoring, and updating of GenAI microservice chains, such as those in [GenAIExamples](https://github.com/opea-project/GenAIExamples) on Kubernetes. It essentially supports a Kubernetes Custom Resource Definition for GenAI chains/pipelines that may be comprised of sequential, conditional, and parallel steps. |
| `kubernetes-addons`       | Deploy Kubernetes add-ons for OPEA.                                                                                                                                                                                                                                                                                                                                                |
| `proxy`                   | OPEA Pipeline Proxy is an enhancement of the default Istio proxy with additional features designed specifically for OPEA RAG pipelines.                                                                                                                                                                                                                                            |
| `scripts`                 | Scripts for testing, tools to facilitate OPEA deployment, and etc.                                                                                                                                                                                                                                                                                                                 |

## Prerequisite

GenAIInfra uses Kubernetes as the cloud native infrastructure. Follow the steps below to prepare the Kubernetes environment.

### Setup Kubernetes cluster

Follow [Kubernetes official setup guide](https://kubernetes.io/docs/setup/) to setup Kubernetes. We recommend to use Kubernetes with version >= 1.27.

There are different methods to setup Kubernetes production cluster, such as [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/), [kubespray](https://kubespray.io/), and [more](https://kubernetes.io/docs/setup/production-environment/tools/).

NOTE: We recommend to use containerd when choosing the container runtime during Kubernetes setup. Docker engine is also verified on Ubuntu 22.04 and above.

### (Optional) To run GenAIInfra on [Intel Gaudi](https://habana.ai/products/) product

The following steps are optional. They're only required if you want to run the workloads on Intel Gaudi product.

1. Check the [support matrix](https://docs.habana.ai/en/latest/Support_Matrix/Support_Matrix.html) to make sure that environment meets the requirements.

2. [Install Intel Gaudi software stack](https://docs.habana.ai/en/latest/Installation_Guide/Bare_Metal_Fresh_OS.html#driver-fw-install-bare).

3. [Install and setup container runtime](https://docs.habana.ai/en/latest/Installation_Guide/Bare_Metal_Fresh_OS.html#set-up-container-usage), based on the container runtime used by Kubernetes.

   NOTE: Make sure you configure the appropriate container runtime based on the type of container runtime you installed during Kubernetes setup.

4. [Install Intel Gaudi device plugin for Kubernetes](https://docs.habana.ai/en/latest/Installation_Guide/Additional_Installation/Kubernetes_Installation/index.html).

Alternatively, Intel provides a [base operator](https://docs.habana.ai/en/latest/Installation_Guide/Additional_Installation/Kubernetes_Installation/Kubernetes_Operator.html) to manage the Gaudi software stack.

## Usages

### Use GenAI Microservices Connector (GMC) to deploy and adjust GenAIExamples

Follow [GMC README](microservices-connector/README.md)
to install GMC into your kubernetes cluster. [GenAIExamples](https://github.com/opea-project/GenAIExamples) contains several sample GenAI example use case pipelines such as ChatQnA, DocSum, etc.
Once you have deployed GMC in your Kubernetes cluster, you can deploy any of the example pipelines by following its Readme file (e.g. [Docsum](/GenAIExamples/DocSum/kubernetes/gmc/README.md)).

### Use helm charts to deploy

To deploy GenAIExamples to Kubernetes using helm charts, you need [Helm](https://helm.sh/docs/intro/install/) installed on your machine.

For a detailed version, see [Deploy GenAIExample/GenAIComps using helm charts](helm-charts/README.md)

### Use terraform to deploy on cloud service providers

You can use [Terraform](https://www.terraform.io/) to create infrastructure to run OPEA applications on various cloud service provider (CSP) environments.

- [AWS/EKS: Create managed Kubernetes cluster on AWS for OPEA](cloud-service-provider/aws/eks/terraform/README.md)
- [Azure/AKS: Create managed Kubernetes cluster on Azure for OPEA](cloud-service-provider/azure/aks/terraform/README.md)
- [GCP/GKE: Create managed Kubernetes cluster on GCP for OPEA](cloud-service-provider/gcp/gke/terraform/README.md)

## Additional Content

- [Code of Conduct](/community/CODE_OF_CONDUCT.md)
- [Contribution](/community/CONTRIBUTING.md)
- [Security Policy](/community/SECURITY.md)
- [Legal Information](LEGAL_INFORMATION.md)
