---
title: "[k8s] OKD Virtualization Automate VMs with Tekton Pipelines - part 2"
date: 2023-12-11
categories: [ngoprek, server, cloud, kubernetes, openshift, okd]
tags:
  - Jekyll
  - update
---

### OKD Virtualization and Tekton Pipelines
Let's continue discussing OKD Virtualization. Now, I'll show you how we can automate the setup and customization of VMs in OKD Virtualization using Tekton Pipelines.

### Tekton Pipeline
Tekton is an open-source cloud-native Continuous Integration and Continuous Delivery/Deployment (CI/CD) solution. It simplifies deployments across various platforms by abstracting the underlying implementation details.

Tekton introduces several standard Custom Resource Definitions (CRDs) for defining CI/CD pipelines that can be used across different Kubernetes distributions.

### How Tekton Works?

![image11](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/okd11.png)

- Task: It defines a set of build steps, such as compiling code, running tests, and building and deploying images.
- Pipeline: It defines the set of tasks that make up a pipeline.
- PipelineResource: It defines an object that serves as an input (e.g., a Git repository) or an output (e.g., a Docker image) of the pipeline.
- PipelineRun: It instantiates a Pipeline for execution with specific inputs, outputs, and execution parameters.

### Kubevirt Tekton tasks
KubeVirt Tekton tasks offer specific Tekton tasks designed for OpenShift Virtualization. These tasks primarily focus on:
- Creating, updating, and managing KubeVirt resources (such as VMs, DataVolumes, DataSources, and Templates).
- Performing operations on disk images using libguestfs tools.

List of Kubevirt Tekton tasks:
- Create Virtual Machines
  - create-vm-from-manifest - create VM from yaml manifest
  - create-vm-from-template - create VM from template
- Utilize Templates
  - copy-template - copy template
  - modify-vm-template - update template metadata
- Create DataVolumes/DataSources
  - create-datavolume-from-manifest (4.11) - create dataVolume
  - modify-data-object (>= 4.12) - create/delete dataVolume/dataSource
- Generate SSH Keys
  - generate-ssh-keys - generate SSH keys and store them in cluster
- Execute Commands in Virtual Machines
  - execute-in-vm: execute commands over SSH
  - cleanup-vm: execute commands and/or stop/delete VMs
- Manipulate PVCs with libguestfs tools
  - disk-virt-customize: execute virt-customize commands in PVCs
  - disk-virt-sysprep: execute virt-sysprep commands in PVCs
- Wait for Virtual Machine Instance Status
  - wait-for-vmi-status - wait for VM status

[https://github.com/kubevirt/kubevirt-tekton-tasks](https://github.com/kubevirt/kubevirt-tekton-tasks)

### Setup Tekton Tasks Operator 

install tekton operator
```
$ export TEKTON_VERSION=v0.65.1
$ oc apply -f  "https://github.com/tektoncd/operator/releases/download
/${TEKTON_VERSION}/openshift-release.yaml"
```

enable tekton task from okd hco
```
$ oc patch hco kubevirt-hyperconverged -n kubevirt-hyperconverged --type=merge -p '{"spec":{"featureGates":{"deployTektonTaskResources":true}}}'
```

### Create Tekton Pipelines
source: [https://github.com/ammarun11/kubevirt-tekton-pipelines](https://github.com/ammarun11/kubevirt-tekton-pipelines)

We'll create pipelines for Ubuntu VMs and Windows VMs
Go to terminal or oc/kubectl client, 
```

oc create ns openinfra2023

oc project openinfra2023

# Ubuntu VM piplines
oc create -f https://raw.githubusercontent.com/ammarun11/kubevirt-tekton-pipelines/main/examples/pipelines/okd/server-deployer-flask/server-deployer-flask-ubuntu.yaml

# Windows VM pipelines
## Need Create ConfigMap unattend & sysprep for automate deployment for windows image

oc create -f https://raw.githubusercontent.com/ammarun11/kubevirt-tekton-pipelines/main/examples/pipelines/okd/windows10/windows10-customize-sysprep.yaml

oc create -f https://raw.githubusercontent.com/ammarun11/kubevirt-tekton-pipelines/main/examples/pipelines/okd/windows10/windows10-template-autounattend.yaml

# Windows VM Template
oc create -f https://raw.githubusercontent.com/ammarun11/kubevirt-tekton-pipelines/main/examples/pipelines/okd/windows10/windows10-template.yaml

# Windows VM Customize
oc create -f https://raw.githubusercontent.com/ammarun11/kubevirt-tekton-pipelines/main/examples/pipelines/okd/windows10/windows10-customize.yaml

```

Results,
![image12](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/okd12.png)

![image13](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/okd13.png)

For creating VMs will be on next part,
[Next, Part 3](https://ammarun.my.id/ngoprek/server/cloud/kubernetes/openshift/okd/OKD-virtualization-part3/)

## Reference
-   [OKD Docs](https://docs.okd.io/4.13/virt/about-virt.html)
