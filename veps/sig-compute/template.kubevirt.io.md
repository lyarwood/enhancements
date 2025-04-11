# VEP #NNNN: Native Support For VirtualMachine Templates

## Release Signoff Checklist

Items marked with (R) are required *prior to targeting to a milestone / release*.

- [ ] (R) Enhancement issue created, which links to VEP dir in [kubevirt/enhancements] (not the initial VEP PR)

## Overview

Virtual Machine templates are traditionally pre-configured virtual machines that serve as blueprints for creating new VMs. They encapsulate the operating system, installed software, and configuration settings, allowing for the rapid and consistent deployment of virtual machines. Using templates streamlines the VM creation process, reduces errors, and ensures uniformity across the virtualized environment.

While KubeVirt provides many of the building blocks associated with VM templates such as snapshots, import/export, and cloning of VirtualMachines, there is no high-level, easy-to-use workflow available to end-users. This enhancement aims to provide such a workflow, reusing these existing building blocks where possible.

## Motivation

The concept of in-cluster templating is generally discouraged by Kubernetes, which instead focuses on external templating of object definitions through tooling such as Helm, Kustomize, etc.

However virtual machines

Downstream vendors provide limited support for in-cluster VirtualMachine templating. For example, OKD and OpenShift provide their own template CRD that is used to provide golden image based VirtualMachine templates to end users through the common-templates project. There is currently no support in OKD or OpenShift for existing VirtualMachines to be turned into reusable templates with the same level of parametrisation and customisation as the golden image templates.

## Use cases

## Goals

- Provide native support for in-cluster templating of golden image based VirtualMachines
- Provide native support for in-cluster templating of existing VirtualMachines
- Provide inter-namespace sharing of templated VirtualMachines
- Provide inter-cluster sharing of templated VirtualMachines

## Non Goals

## Definition of users

## User Stories

- As a VirtualMachine owner I would like to create a VirtualMachine from a native in-cluster template that
  - can be hosted in a separate namespace
  - tracks a periodically updated golden image
  - was originally created from a VirtualMachine that had customisation
  - can customise and anonymise certain aspects of the VirtualMachine
- As a VirtualMachine owner I would like to share my native in-cluster templates between namespaces I control
- As a VirtualMachine owner I would like to import and export my native in-cluster templates between clusters into namespaces that I control

## Repos

- kubevirt/kubevirt
- kubevirt/common-templates

## Design

### template.kubevirt.io/v1alpha1

A new `template.kubevirt.io` APIGroup will be introduced with an initial `v1alpha1` version being provided.

#### VirtualMachineTemplate

A new namespaced VirtualMachineTemplate CRD will be introduced within this APIGroup with initial limited support for the golden image based template workflow where a VirtualMachine definition is statically defined within the template itself:

```yaml
apiVersion: template.kubevirt.io/v1alpha1
kind: VirtualMachineTemplate
metadata:
  name: fedora
  namespace: templates
spec:
  virtualMachine:
    metadata:
      name: fedora 
    spec:
      instancetype:
        name: u1.medium
      preference:
        name: fedora
```

#### clone.kubevirt.io/v1beta1

The clone.kubevirt.io APIGroup and VirtualMachineClone CRD it provides will be extended to support using a VirtualMachineTemplate as a source when a VirtualMachine is the target.

```yaml
apiVersion: clone.kubevirt.io/v1beta1
kind: VirtualMachineClone
metadata:
  name: clone
spec:
  source:
    apiGroup: template.kubevirt.io
    kind: VirtualMachineTemplate
    name: fedora
  target:
    apiGroup: kubevirt.io
    kind: VirtualMachine
    name: my-fedora-vm-from-template
```

This support will initially be within the same namespace as the template as we explore options around extending the APIGroup and CRD to support cross-namespace sources and targets.

### Template using an existing VirtualMachine

#### VirtualMachineTemplate

```yaml
apiVersion: template.kubevirt.io/v1alpha2
kind: VirtualMachineTemplate
metadata:
  name: fedora
  namespace: templates
spec:
  virtualMachineRef:
    name: fedora
    namespace: my-namespace
```

```yaml
apiVersion: template.kubevirt.io/v1alpha2
kind: VirtualMachineTemplate
metadata:
  name: fedora
  namespace: templates
spec:
  virtualMachineRef:
    name: fedora
    namespace: my-namespace
status:
  virtualMachineSnapshotRef:
    name: template-snapshot
    namespace: my-namespace
```

## API Examples

## Alternatives

### Use VirtualMachineSnapshots for both use cases

## Scalability

## Update/Rollback Compatibility

## Functional Testing Approach

## Implementation Phases

## Feature lifecycle Phases

### Alpha

#### template.kubevirt.io/v1alpha1 (v1.7.0)

### Beta

#### template.kubevirt.io/v1beta1 (>=v1.8.0)

### GA

#### template.kubevirt.io/v1 (>=v1.9.0)
