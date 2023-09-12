# Deploy RHACM, GitOps to OCP using Ansible Automation Platform

## Prerequisites

- A Hub OpenShift cluster
- Red Hat Registry Pull Secret (or an offline mirror registry pull secret)
- SSH Key pair
- A RH Ansible Automation Platform subscription packaged as a Satellite-compatible zip'd manifest file

## Running Locally

- Install Ansible on a local host
- Install needed pip modules: `pip3 install -r ./requirements.txt`
- Install needed Ansible Collections: `ansible-galaxy collection install -r ./collections/requirements.yml`
- Log into Hub OpenShift with `oc login`

## OpenShift Hub Cluster Setup

### 1. Setup the Hub Cluster - Deploy Operators & Workloads

The following Playbook will take a fresh OpenShift 4.9+ cluster and deploy:

- Red Hat Advanced Cluster Management
- Red Hat GitOps (ArgoCD)

Though by default nothing is deployed, you must specify workloads to enable:

```bash
ansible-playbook 1_deploy.yaml \
  -e deploy_rhacm=true \
  -e enable_rhacm_observability=true \
  -e deploy_rh_gitops=true \
```

Or use a variable file, such as `deployment.vars.yaml`:

```yaml
---
deploy_reflector: true

deploy_rhacm: true
enable_rhacm_observability: true
deploy_rh_gitops: true

```

And then call the Playbook with the additional variable file:

```bash
ansible-playbook -e "@deployment.vars.yaml" 1_deploy.yaml
```

### 2. Configure the Hub Cluster - Configure Operators & Workloads

The configuration playbook will do the following:

- Configure AAP2 Controller with a new Organization, Application, Credentials, Inventory, Project, Job Templates, and RBAC
- Configure Red Hat GitOps (ArgoCD) with a set of Projects, an Application, Git Repo with Credentials, and RBAC

```bash
ansible-playbook 2_configure.yaml \
  -e configure_rhacm=true \
  -e configure_aap2_controller=true \
  -e configure_rh_gitops=true \
  -e pull_secret_path="~/rh-ocp-pull-secret.json" \
  # ...
```

Or, use a variable file, such as `configuration.vars.yaml`:

```yaml
---
configure_rhacm: true
configure_aap2_controller: true
configure_rh_gitops: true

use_ztp_mirror: true
use_services_not_routes: true

pull_secret_path: ~/rh-ocp-pull-secret.json

## View the default variables defined in the `2_configure.yaml` playbook for more control over SCM and other configuration
```

And then call the Playbook with the additional variable file:

```bash
ansible-playbook -e "@configuration.vars.yaml" 2_configure.yaml
```

> You can find an all the credential variables and their information in the `example_vars/example_create_credentials.vars.yaml` file.

