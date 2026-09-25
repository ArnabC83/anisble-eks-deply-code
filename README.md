# Deploy NGINX to Amazon EKS

This Ansible playbook connects to an existing Amazon EKS cluster, creates an
NGINX Deployment with two replicas, and exposes it through a Kubernetes
LoadBalancer Service.

## Prerequisites

- An existing EKS cluster
- AWS CLI configured with credentials that can describe the cluster and update kubeconfig
- Ansible
- Python 3 and the Kubernetes Python client

Install the Ansible collection and Python dependencies:

```bash
ansible-galaxy collection install kubernetes.core
python3 -m pip install kubernetes PyYAML pytest
```

The AWS identity must also have permission to access the EKS cluster and the
Kubernetes identity mapped to that AWS identity must be allowed to create
Deployments and Services in the target namespace.

## Configure AWS access

Verify that the AWS CLI is using the expected account and region:

```bash
aws sts get-caller-identity
aws eks describe-cluster --region us-east-1 --name my-eks-cluster
```

The default values in the playbook target `us-east-1` and a cluster named
`my-eks-cluster`. Override them with Ansible extra variables when needed.

## Run the deployment

From this directory, run:

```bash
ansible-playbook plybook.yaml \
  -e aws_region=us-east-1 \
  -e eks_cluster_name=my-eks-cluster \
  -e kubeconfig_path="$HOME/.kube/config"
```

The playbook first runs `aws eks update-kubeconfig`, then creates or updates:

- Deployment: `nginx-deployment`
- Service: `nginx-service`

Check the rollout and retrieve the external load balancer address:

```bash
kubectl --kubeconfig "$HOME/.kube/config" rollout status deployment/nginx-deployment
kubectl --kubeconfig "$HOME/.kube/config" get pods,service nginx-service
```

## Customize the deployment

Any of these values can be overridden with `-e`:

```bash
ansible-playbook plybook.yaml \
  -e aws_region=us-west-2 \
  -e eks_cluster_name=production \
  -e app_name=web \
  -e replica_count=3 \
  -e service_port=8080 \
  -e kubeconfig_path="$HOME/.kube/config"
```

## Run the tests

The test suite parses the playbook and validates the Deployment and Service
configuration. It does not connect to AWS or Kubernetes.

```bash
pytest -q tests/test_playbook.py
```