# ansible-builder-shipwright

Creation of Ansible execution environments using [Ansible Builder](https://github.com/ansible/ansible-builder) and [Shipwright](https://github.com/ansible/ansible-builder)

## Overview

Shipwright is a framework for building container images within Kubernetes. Ansible Execution environments are container images that are produced through the Ansible Builder project to enable the execution of Ansible automation using [ansible-runner](https://github.com/ansible/ansible-runner). Builds are achieved by creating a custom Shipwright `ClusterBuildStrategy` with the logic to produce container based Ansible Execution environments which are then published to a container registry. 

## Installation

Use the following steps to install Shipwright and the `ClusterBuildStrategy` to your environment

1. Install Tekton/OpenShift Pipelines. In an OpenShift environment, this can be achieved in OperatorHub by installing either OpenShift Pipelines or OpenShift GitOps as it includes OpenShift Pipelines.
2. Install Shipwright

```
git clone https://github.com/shipwright-io/build
cd build
git checkout -b v0.3.0 tags/v0.3.0
make install
```

Patch the Operator image with the appropriate tagged version

```
kubectl patch deployment build-operator  --type json   -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"quay.io/shipwright/shipwright-operator:v0.3.0"}]'
```

Confirm the operator is running in the `build-operator` project

```
kubectl get pods -n build-operator
```

3. Install the `ansible-builder` `ClusterBuildStrategy` by cloning this repository and adding the `ClusterBuildStrategy` to your environment

```
git clone https://github.com/sabre1041/ansible-builder-shipwright
cd ansible-builder-shipwright
kubectl apply -f resources/ansible-builder-clusterbuildpolicy.yaml
```

## Example

To demonstrate how an Ansible Execution environment can be produced and published to a container registry using Shipwright, an example implementation is provided within this repository.

1. Create a new Namespace called `ansible-builder-shipwright`

```
kubectl create namespace `ansible-builder-shipwright`
```

2. Register the Shipwright Build

```
kubectl create -f example/ansible-builder-build.yaml
```

Confirm that the Build has been registered properly

```
kubectl get builds.build.dev ansible-builder-example

NAME                                        REGISTERED   REASON                        BUILDSTRATEGYKIND      BUILDSTRATEGYNAME   CREATIONTIME
ansible-builder-example                     False        RemoteRepositoryUnreachable   ClusterBuildStrategy   ansible-builder     54s
ansible-builder-example                     True         Succeeded                     ClusterBuildStrategy   ansible-builder     4h32m
```

3. Grant Access to the `privileged` SCC (OpenShift)

Elevated permissions through the use of a privileged container is needed during the build process and when running on OpenShift, access to the `privileged` Security Context Constraint is required. Grant the `pipelines` service account access to the SCC by executing the following command:

```
oc adm policy add-scc-to-user privileged -z pipeline
```

3. Start a new Build by creating a `BuildRun`

```
kubectl create -f example/ansible-builder-buildrun.yaml
```

4. Monitor the progress of the BuildRun

You can monitor the progress of the build which is executed using a Tekton `TaskRun` if you have the Tekton CLI (`tkn`) installed on your machine

```
tkn taskrun logs -f -L
```

The above command will display the logs for the most recent TaskRun

Once the build is complete, an image will be published to OpenShift's internal image registry. Alternatively, the produced image can also be stored in an external registry, such as quay.io by modifying the `ansible-builder-example` Build resource and modifying the `output.image` field. Credentials to authenticate with the external registry will most likely need to be provided. Consult the Shipwright documentation with the steps needed to configure.