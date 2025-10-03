# Custom Resource Definitions (CRDs) in Kubernetes

## Overview

Custom Resource Definitions (CRDs) are a powerful Kubernetes feature that allows users to extend the Kubernetes API with their own custom resources. CRDs enable the creation of domain-specific APIs and objects that can be managed using standard Kubernetes tools and workflows.

### What are CRDs?

CRDs are Kubernetes resources that:
- Define new custom resource types
- Extend the Kubernetes API
- Enable declarative configuration of custom objects
- Work with standard Kubernetes features like kubectl

### Key Features

1. **API Extension**: Extend Kubernetes API with custom resources
2. **Declarative Management**: Use YAML manifests to define custom resources
3. **Validation**: Validate custom resource specifications
4. **Storage**: Store custom resource data in etcd
5. **Integration**: Work with controllers and operators

## Architecture

### Core Components

#### 1. Custom Resource Definition (CRD)

The CRD defines:
- API group and version
- Resource kind and names
- Schema validation
- Scope (cluster or namespaced)
- Subresources

#### 2. Custom Resource (CR)

The Custom Resource is:
- An instance of a CRD
- Managed like any Kubernetes object
- Stored in etcd
- Subject to validation and admission control

#### 3. Controller/Operator

The controller/operator:
- Watches for CR changes
- Implements reconciliation logic
- Manages resource lifecycle
- Handles error conditions

### Architecture Diagram

```
+-------------------+     +-------------------+     +-------------------+
| User/Developer    |     | Kubernetes API    |     | etcd             |
| +---------------+ |     | Server           | |     | +---------------+ |
| | kubectl apply  | |     | +---------------+ |     | | CRD Storage    | |
| | YAML Manifests | |     | | CRD Validation | |     | | CR Storage    | |
| +---------------+ |     | | Admission Ctrl | |     | +---------------+ |
+-------------------+     | +---------------+ |     +-------------------+
          |                     |                       |
          +---------------------+-----------------------+
                                  |
                      +-------------------+
                      | Controller/      |
                      | Operator          |
                      | +---------------+ |
                      | | Reconciliation | |
                      | | Business Logic | |
                      | | Resource Mgmt  | |
                      | +---------------+ |
                      +-------------------+
```

## Core Concepts

### 1. CRD Definition

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
                enum: [mysql, postgresql, mongodb]
                default: mysql
              version:
                type: string
                default: "8.0"
              replicas:
                type: integer
                minimum: 1
                maximum: 10
                default: 3
              storageSize:
                type: string
                pattern: '^[0-9]+(Gi|Mi|Ti)$'
                default: "10Gi"
          status:
            type: object
            properties:
              phase:
                type: string
                enum: [Pending, Creating, Ready, Failed]
              message:
                type: string
              readyReplicas:
                type: integer
    additionalPrinterColumns:
    - name: Engine
      type: string
      jsonPath: .spec.engine
    - name: Replicas
      type: integer
      jsonPath: .spec.replicas
    - name: Phase
      type: string
      jsonPath: .status.phase
  scope: Namespaced
  names:
    plural: databases
    singular: database
    kind: Database
    shortNames:
    - db
```

### 2. Custom Resource Instance

```yaml
apiVersion: example.com/v1
kind: Database
metadata:
  name: my-database
  namespace: default
spec:
  engine: postgresql
  version: "13.4"
  replicas: 3
  storageSize: "20Gi"
```

### 3. CRD with Validation

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: tenants.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        required: ["spec"]
        properties:
          spec:
            type: object
            required: ["name", "email"]
            properties:
              name:
                type: string
                minLength: 1
                maxLength: 63
                pattern: '^[a-z0-9]([-a-z0-9]*[a-z0-9])?$'
              email:
                type: string
                format: email
              quota:
                type: object
                properties:
                  cpu:
                    type: string
                    pattern: '^[0-9]+(m)?$'
                  memory:
                    type: string
                    pattern: '^[0-9]+(Gi|Mi|Ti)?$'
                  storage:
                    type: string
                    pattern: '^[0-9]+(Gi|Mi|Ti)?$'
              resources:
                type: object
                properties:
                  databases:
                    type: integer
                    minimum: 0
                    maximum: 100
                  users:
                    type: integer
                    minimum: 1
                    maximum: 1000
          status:
            type: object
            properties:
              phase:
                type: string
                enum: [Active, Suspended, Terminated]
              createdAt:
                type: string
                format: date-time
              lastModified:
                type: string
                format: date-time
  scope: Namespaced
  names:
    plural: tenants
    singular: tenant
    kind: Tenant
    shortNames:
    - tenant
```

### 4. CRD with Subresources

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: backups.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              sourceDatabase:
                type: string
              schedule:
                type: string
                pattern: '^[0-9]+\s+[0-9]+\s+\*\s+\*\s+\*$'
              retention:
                type: integer
                minimum: 1
                maximum: 365
          status:
            type: object
            properties:
              lastBackupTime:
                type: string
                format: date-time
              nextBackupTime:
                type: string
                format: date-time
              backupCount:
                type: integer
    subresources:
      status: {}
      scale:
        specReplicasPath: .spec.replicas
        statusReplicasPath: .status.replicas
  scope: Namespaced
  names:
    plural: backups
    singular: backup
    kind: Backup
```

## Configuration Examples

### 1. Simple CRD for Application Configuration

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: appconfigs.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              appName:
                type: string
              version:
                type: string
              replicas:
                type: integer
                minimum: 1
                maximum: 10
              image:
                type: string
              ports:
                type: array
                items:
                  type: object
                  properties:
                    containerPort:
                      type: integer
                    protocol:
                      type: string
                      enum: [TCP, UDP]
                    name:
                      type: string
          status:
            type: object
            properties:
              availableReplicas:
                type: integer
              conditions:
                type: array
                items:
                  type: object
                  properties:
                    type:
                      type: string
                    status:
                      type: string
                    reason:
                      type: string
                    message:
                      type: string
                    lastUpdateTime:
                      type: string
                      format: date-time
  scope: Namespaced
  names:
    plural: appconfigs
    singular: appconfig
    kind: AppConfig
```

### 2. CRD for Complex Infrastructure

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: clusters.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required: ["provider", "region", "nodePools"]
            properties:
              provider:
                type: string
                enum: [aws, gcp, azure]
              region:
                type: string
              version:
                type: string
                default: "1.25.0"
              network:
                type: object
                properties:
                  vpcCIDR:
                    type: string
                    pattern: '^([0-9]{1,3}\.){3}[0-9]{1,3}/[0-9]{1,2}$'
                  podCIDR:
                    type: string
                    pattern: '^([0-9]{1,3}\.){3}[0-9]{1,3}/[0-9]{1,2}$'
                  serviceCIDR:
                    type: string
                    pattern: '^([0-9]{1,3}\.){3}[0-9]{1,3}/[0-9]{1,2}$'
              nodePools:
                type: array
                items:
                  type: object
                  required: ["name", "instanceType", "minSize", "maxSize"]
                  properties:
                    name:
                      type: string
                    instanceType:
                      type: string
                    minSize:
                      type: integer
                      minimum: 1
                    maxSize:
                      type: integer
                      minimum: 1
                    labels:
                      type: object
                      additionalProperties: true
                    taints:
                      type: array
                      items:
                        type: object
                        properties:
                          key:
                            type: string
                          value:
                            type: string
                          effect:
                            type: string
                            enum: [NoSchedule, PreferNoSchedule, NoExecute]
                    tags:
                      type: object
                      additionalProperties: true
          status:
            type: object
            properties:
              phase:
                type: string
                enum: [Provisioning, Ready, Updating, Deleting, Failed]
              endpoint:
                type: string
              createdAt:
                type: string
                format: date-time
              updatedAt:
                type: string
                format: date-time
              conditions:
                type: array
                items:
                  type: object
                  properties:
                    type:
                      type: string
                    status:
                      type: string
                    reason:
                      type: string
                    message:
                      type: string
                    lastTransitionTime:
                      type: string
                      format: date-time
  scope: Cluster
  names:
    plural: clusters
    singular: cluster
    kind: Cluster
    shortNames:
    - cluster
```

### 3. CRD with Versioning

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: pipelines.example.com
spec:
  group: example.com
  versions:
  - name: v1alpha1
    served: true
    storage: false
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              name:
                type: string
              stages:
                type: array
                items:
                  type: object
                  properties:
                    name:
                      type: string
                    type:
                      type: string
                      enum: [build, test, deploy]
                    image:
                      type: string
                    command:
                      type: array
                      items:
                        type: string
          status:
            type: object
            properties:
              phase:
                type: string
              currentStage:
                type: string
              startTime:
                type: string
                format: date-time
              completionTime:
                type: string
                format: date-time
  - name: v1beta1
    served: true
    storage: false
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              metadata:
                type: object
                properties:
                  name:
                    type: string
                  description:
                    type: string
                  labels:
                    type: object
                    additionalProperties: true
              pipeline:
                type: object
                properties:
                  stages:
                    type: array
                    items:
                      type: object
                      properties:
                        name:
                          type: string
                        type:
                          type: string
                          enum: [build, test, deploy, security, approval]
                        image:
                          type: string
                        command:
                          type: array
                          items:
                            type: string
                        args:
                          type: array
                          items:
                            type: string
                        env:
                          type: array
                          items:
                            type: object
                            properties:
                              name:
                                type: string
                              value:
                                type: string
                        timeout:
                          type: string
                          pattern: '^[0-9]+(s|m|h)$'
                        resources:
                          type: object
                          properties:
                            requests:
                              type: object
                              properties:
                                cpu:
                                  type: string
                                memory:
                                  type: string
                            limits:
                              type: object
                              properties:
                                cpu:
                                  type: string
                                memory:
                                  type: string
          status:
            type: object
            properties:
              phase:
                type: string
                enum: [Pending, Running, Succeeded, Failed, Cancelled]
              startTime:
                type: string
                format: date-time
              completionTime:
                type: string
                format: date-time
              stages:
                type: array
                items:
                  type: object
                  properties:
                    name:
                      type: string
                    phase:
                      type: string
                      enum: [Pending, Running, Succeeded, Failed, Skipped]
                    startTime:
                      type: string
                      format: date-time
                    completionTime:
                      type: string
                      format: date-time
                    message:
                      type: string
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required: ["pipeline"]
            properties:
              metadata:
                type: object
                properties:
                  name:
                    type: string
                  description:
                    type: string
                  labels:
                    type: object
                    additionalProperties: true
                  annotations:
                    type: object
                    additionalProperties: true
              pipeline:
                type: object
                required: ["stages"]
                properties:
                  timeout:
                    type: string
                    pattern: '^[0-9]+(s|m|h)$'
                  stages:
                    type: array
                    items:
                      type: object
                      required: ["name", "type"]
                      properties:
                        name:
                          type: string
                        type:
                          type: string
                          enum: [build, test, deploy, security, approval, notify]
                        description:
                          type: string
                        image:
                          type: string
                        command:
                          type: array
                          items:
                            type: string
                        args:
                          type: array
                          items:
                            type: string
                        env:
                          type: array
                          items:
                            type: object
                            properties:
                              name:
                                type: string
                              value:
                                type: string
                        timeout:
                          type: string
                          pattern: '^[0-9]+(s|m|h)$'
                        resources:
                          type: object
                          properties:
                            requests:
                              type: object
                              properties:
                                cpu:
                                  type: string
                                memory:
                                  type: string
                            limits:
                              type: object
                              properties:
                                cpu:
                                  type: string
                                memory:
                                  type: string
                        when:
                          type: string
                        continueOnError:
                          type: boolean
                        retries:
                          type: integer
                          minimum: 0
                          maximum: 10
          status:
            type: object
            properties:
              phase:
                type: string
                enum: [Pending, Running, Succeeded, Failed, Cancelled, Timeout]
              startTime:
                type: string
                format: date-time
              completionTime:
                type: string
                format: date-time
              stages:
                type: array
                items:
                  type: object
                  properties:
                    name:
                      type: string
                    phase:
                      type: string
                      enum: [Pending, Running, Succeeded, Failed, Skipped, Timeout]
                    startTime:
                      type: string
                      format: date-time
                    completionTime:
                      type: string
                      format: date-time
                    message:
                      type: string
                    duration:
                      type: string
                    retries:
                      type: integer
              conditions:
                type: array
                items:
                  type: object
                  properties:
                    type:
                      type: string
                    status:
                      type: string
                    reason:
                      type: string
                    message:
                      type: string
                    lastTransitionTime:
                      type: string
                      format: date-time
  scope: Namespaced
  names:
    plural: pipelines
    singular: pipeline
    kind: Pipeline
    shortNames:
    - pipeline
```

## Interview Questions

### Beginner Level

1. **What are Custom Resource Definitions (CRDs) and what problem do they solve?**
   - CRDs extend the Kubernetes API with custom resource types, solving the problem of needing domain-specific objects that can be managed using standard Kubernetes tools.

2. **What are the main components of a CRD?**
   - The main components are the CRD definition (schema, validation, scope), custom resource instances, and controllers/operators that manage the resources.

3. **What is the difference between a CRD and a ConfigMap?**
   - CRDs define new API objects with validation and schema, while ConfigMaps store configuration data as key-value pairs without validation.

4. **How do you create and apply a CRD?**
   - Create a YAML manifest defining the CRD, then apply it using `kubectl apply -f crd.yaml`. After that, you can create instances of the custom resource.

### Intermediate Level

5. **What is OpenAPI schema validation in CRDs?**
   - OpenAPI schema validation defines the structure, types, and constraints for custom resources, ensuring they conform to the defined specification.

6. **How do CRDs work with controllers and operators?**
   - Controllers watch for changes to custom resources and implement reconciliation logic to maintain the desired state, while operators are more sophisticated controllers that manage complex applications.

7. **What are the different scopes of CRDs?**
   - CRDs can be cluster-scoped (available across all namespaces) or namespaced (available only within a specific namespace).

8. **How do you handle versioning with CRDs?**
   - CRDs support multiple versions, allowing for API evolution with conversion between versions and deprecation strategies.

### Advanced Level

9. **How would you implement a complex CRD with multiple versions and conversion?**
   - Define multiple versions in the CRD, implement conversion webhooks if needed, and ensure backward compatibility with proper validation and default values.

10. **What are the security implications of using CRDs in production?**
    - Consider RBAC permissions, admission control, resource quotas, and the potential impact of custom resources on cluster security and stability.

11. **How do you handle CRD upgrades and migrations?**
    - Implement versioned CRDs, use conversion webhooks, create migration controllers, and test thoroughly in non-production environments.

12. **What are the best practices for designing CRDs for enterprise applications?**
    - Design clear APIs, use proper validation, implement comprehensive status reporting, provide good documentation, and ensure backward compatibility.

## Best Practices

### 1. CRD Design Best Practices

```yaml
# Well-designed CRD with comprehensive validation
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: workloads.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        required: ["spec"]
        properties:
          spec:
            type: object
            required: ["name", "type"]
            properties:
              name:
                type: string
                description: "Name of the workload"
                minLength: 1
                maxLength: 63
                pattern: '^[a-z0-9]([-a-z0-9]*[a-z0-9])?$'
              type:
                type: string
                description: "Type of workload"
                enum: [deployment, statefulset, daemonset, job, cronjob]
              replicas:
                type: integer
                description: "Number of replicas"
                minimum: 0
                maximum: 100
                default: 1
              image:
                type: string
                description: "Container image"
                pattern: '^[a-z0-9]+([._-][a-z0-9]+)*(/[a-z0-9]+([._-][a-z0-9]+)*)*(:[a-z0-9]+([._-][a-z0-9]+)*)?$'
              ports:
                type: array
                description: "Container ports"
                items:
                  type: object
                  properties:
                    containerPort:
                      type: integer
                      description: "Container port number"
                      minimum: 1
                      maximum: 65535
                    protocol:
                      type: string
                      description: "Protocol"
                      enum: [TCP, UDP]
                      default: TCP
                    name:
                      type: string
                      description: "Port name"
                      maxLength: 15
                      pattern: '^[a-z0-9]([-a-z0-9]*[a-z0-9])?$'
              resources:
                type: object
                description: "Resource requirements"
                properties:
                  requests:
                    type: object
                    properties:
                      cpu:
                        type: string
                        description: "CPU request"
                        pattern: '^[0-9]+(\.[0-9]+)?m?$'
                      memory:
                        type: string
                        description: "Memory request"
                        pattern: '^[0-9]+(\.[0-9]+)?(Ki|Mi|Gi|Ti)?$'
                  limits:
                    type: object
                    properties:
                      cpu:
                        type: string
                        description: "CPU limit"
                        pattern: '^[0-9]+(\.[0-9]+)?m?$'
                      memory:
                        type: string
                        description: "Memory limit"
                        pattern: '^[0-9]+(\.[0-9]+)?(Ki|Mi|Gi|Ti)?$'
              env:
                type: array
                description: "Environment variables"
                items:
                  type: object
                  properties:
                    name:
                      type: string
                      description: "Environment variable name"
                      pattern: '^[A-Z_][A-Z0-9_]*$'
                    value:
                      type: string
                      description: "Environment variable value"
                    valueFrom:
                      type: object
                      description: "Source for environment variable value"
                      properties:
                        secretKeyRef:
                          type: object
                          properties:
                            name:
                              type: string
                            key:
                              type: string
                        configMapKeyRef:
                          type: object
                          properties:
                            name:
                              type: string
                            key:
                              type: string
          status:
            type: object
            properties:
              phase:
                type: string
                description: "Current phase of the workload"
                enum: [Pending, Creating, Ready, Updating, Deleting, Failed]
              message:
                type: string
                description: "Status message"
              availableReplicas:
                type: integer
                description: "Number of available replicas"
              conditions:
                type: array
                description: "Conditions representing the status"
                items:
                  type: object
                  properties:
                    type:
                      type: string
                    status:
                      type: string
                    reason:
                      type: string
                    message:
                      type: string
                    lastTransitionTime:
                      type: string
                      format: date-time
    additionalPrinterColumns:
    - name: Type
      type: string
      jsonPath: .spec.type
    - name: Replicas
      type: integer
      jsonPath: .spec.replicas
    - name: Phase
      type: string
      jsonPath: .status.phase
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp
  scope: Namespaced
  names:
    plural: workloads
    singular: workload
    kind: Workload
    shortNames:
    - wl
    categories:
    - all
    - example
```

### 2. CRD Versioning Strategy

```yaml
# CRD with multiple versions and conversion
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: services.example.com
spec:
  group: example.com
  versions:
  - name: v1alpha1
    served: true
    storage: false
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              name:
                type: string
              port:
                type: integer
              protocol:
                type: string
                enum: [TCP, UDP]
  - name: v1beta1
    served: true
    storage: false
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              metadata:
                type: object
                properties:
                  name:
                    type: string
                  labels:
                    type: object
              service:
                type: object
                properties:
                  ports:
                    type: array
                    items:
                      type: object
                      properties:
                        port:
                          type: integer
                        protocol:
                          type: string
                        name:
                          type: string
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            required: ["service"]
            properties:
              metadata:
                type: object
                properties:
                  name:
                    type: string
                  labels:
                    type: object
                  annotations:
                    type: object
              service:
                type: object
                required: ["ports"]
                properties:
                  type:
                    type: string
                    enum: [ClusterIP, NodePort, LoadBalancer, ExternalName]
                  ports:
                    type: array
                    items:
                      type: object
                      required: ["port"]
                      properties:
                        port:
                          type: integer
                          minimum: 1
                          maximum: 65535
                        targetPort:
                          type: integer
                          minimum: 1
                          maximum: 65535
                        protocol:
                          type: string
                          enum: [TCP, UDP, SCTP]
                          default: TCP
                        name:
                          type: string
                          maxLength: 15
                        nodePort:
                          type: integer
                          minimum: 30000
                          maximum: 32767
                  selector:
                    type: object
                  clusterIP:
                    type: string
                  externalName:
                    type: string
          status:
            type: object
            properties:
              clusterIP:
                type: string
              ports:
                type: array
                items:
                  type: object
                  properties:
                    port:
                      type: integer
                    nodePort:
                      type: integer
                    protocol:
                      type: string
              loadBalancer:
                type: object
                properties:
                  ingress:
                    type: array
                    items:
                      type: object
                      properties:
                        ip:
                          type: string
                        hostname:
                          type: string
  conversion:
    strategy: Webhook
    webhook:
      conversionReviewVersions: ["v1", "v1beta1"]
      clientConfig:
        service:
          name: conversion-webhook
          namespace: example-system
          path: "/convert"
        caBundle: <base64-encoded-ca-cert>
  scope: Namespaced
  names:
    plural: services
    singular: service
    kind: Service
    shortNames:
    - svc
```

### 3. Production CRD Implementation

```yaml
# Production-ready CRD with comprehensive features
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: applications.example.com
  annotations:
    api-approved.kubernetes.io: "https://github.com/example/api"
    controller-gen.kubebuilder.io/version: v0.11.0
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        required: ["spec"]
        properties:
          spec:
            type: object
            required: ["name", "components"]
            properties:
              name:
                type: string
                description: "Application name"
                minLength: 1
                maxLength: 63
                pattern: '^[a-z0-9]([-a-z0-9]*[a-z0-9])?$'
              description:
                type: string
                description: "Application description"
                maxLength: 1024
              labels:
                type: object
                description: "Application labels"
                additionalProperties: true
              annotations:
                type: object
                description: "Application annotations"
                additionalProperties: true
              components:
                type: array
                description: "Application components"
                items:
                  type: object
                  required: ["name", "type", "image"]
                  properties:
                    name:
                      type: string
                      description: "Component name"
                      minLength: 1
                      maxLength: 63
                      pattern: '^[a-z0-9]([-a-z0-9]*[a-z0-9])?$'
                    type:
                      type: string
                      description: "Component type"
                      enum: [deployment, statefulset, daemonset, job, cronjob]
                    image:
                      type: string
                      description: "Container image"
                      pattern: '^[a-z0-9]+([._-][a-z0-9]+)*(/[a-z0-9]+([._-][a-z0-9]+)*)*(:[a-z0-9]+([._-][a-z0-9]+)*)?$'
                    replicas:
                      type: integer
                      description: "Number of replicas"
                      minimum: 0
                      maximum: 100
                      default: 1
                    ports:
                      type: array
                      description: "Component ports"
                      items:
                        type: object
                        properties:
                          containerPort:
                            type: integer
                            description: "Container port"
                            minimum: 1
                            maximum: 65535
                          protocol:
                            type: string
                            description: "Protocol"
                            enum: [TCP, UDP, SCTP]
                            default: TCP
                          name:
                            type: string
                            description: "Port name"
                            maxLength: 15
                            pattern: '^[a-z0-9]([-a-z0-9]*[a-z0-9])?$'
                    resources:
                      type: object
                      description: "Resource requirements"
                      properties:
                        requests:
                          type: object
                          properties:
                            cpu:
                              type: string
                              description: "CPU request"
                              pattern: '^[0-9]+(\.[0-9]+)?m?$'
                            memory:
                              type: string
                              description: "Memory request"
                              pattern: '^[0-9]+(\.[0-9]+)?(Ki|Mi|Gi|Ti)?$'
                        limits:
                          type: object
                          properties:
                            cpu:
                              type: string
                              description: "CPU limit"
                              pattern: '^[0-9]+(\.[0-9]+)?m?$'
                            memory:
                              type: string
                              description: "Memory limit"
                              pattern: '^[0-9]+(\.[0-9]+)?(Ki|Mi|Gi|Ti)?$'
                    env:
                      type: array
                      description: "Environment variables"
                      items:
                        type: object
                        properties:
                          name:
                            type: string
                            description: "Environment variable name"
                            pattern: '^[A-Z_][A-Z0-9_]*$'
                          value:
                            type: string
                            description: "Environment variable value"
                          valueFrom:
                            type: object
                            description: "Source for environment variable value"
                            properties:
                              secretKeyRef:
                                type: object
                                properties:
                                  name:
                                    type: string
                                  key:
                                    type: string
                              configMapKeyRef:
                                type: object
                                properties:
                                  name:
                                    type: string
                                  key:
                                    type: string
                    volumes:
                      type: array
                      description: "Component volumes"
                      items:
                        type: object
                        properties:
                          name:
                            type: string
                            description: "Volume name"
                            maxLength: 63
                            pattern: '^[a-z0-9]([-a-z0-9]*[a-z0-9])?$'
                          persistentVolumeClaim:
                            type: object
                            properties:
                              claimName:
                                type: string
                          emptyDir:
                            type: object
                            properties:
                              sizeLimit:
                                type: string
                                pattern: '^[0-9]+(\.[0-9]+)?(Ki|Mi|Gi|Ti)?$'
                          configMap:
                            type: object
                            properties:
                              name:
                                type: string
                          secret:
                            type: object
                            properties:
                              secretName:
                                type: string
                    probes:
                      type: object
                      description: "Health probes"
                      properties:
                        liveness:
                          type: object
                          properties:
                            httpGet:
                              type: object
                              properties:
                                path:
                                  type: string
                                port:
                                  type: integer
                                scheme:
                                  type: string
                                  enum: [HTTP, HTTPS]
                            exec:
                              type: object
                              properties:
                                command:
                                  type: array
                                  items:
                                    type: string
                            tcpSocket:
                              type: object
                              properties:
                                port:
                                  type: integer
                            initialDelaySeconds:
                              type: integer
                              minimum: 0
                            timeoutSeconds:
                              type: integer
                              minimum: 1
                            periodSeconds:
                              type: integer
                              minimum: 1
                            successThreshold:
                              type: integer
                              minimum: 1
                            failureThreshold:
                              type: integer
                              minimum: 1
                        readiness:
                          type: object
                          properties:
                            httpGet:
                              type: object
                              properties:
                                path:
                                  type: string
                                port:
                                  type: integer
                                scheme:
                                  type: string
                                  enum: [HTTP, HTTPS]
                            exec:
                              type: object
                              properties:
                                command:
                                  type: array
                                  items:
                                    type: string
                            tcpSocket:
                              type: object
                              properties:
                                port:
                                  type: integer
                            initialDelaySeconds:
                              type: integer
                              minimum: 0
                            timeoutSeconds:
                              type: integer
                              minimum: 1
                            periodSeconds:
                              type: integer
                              minimum: 1
                            successThreshold:
                              type: integer
                              minimum: 1
                            failureThreshold:
                              type: integer
                              minimum: 1
              dependencies:
                type: array
                description: "Application dependencies"
                items:
                  type: object
                  properties:
                    name:
                      type: string
                      description: "Dependency name"
                    type:
                      type: string
                      description: "Dependency type"
                      enum: [service, database, cache, queue]
                    connection:
                      type: object
                      description: "Connection configuration"
                      properties:
                        host:
                          type: string
                        port:
                          type: integer
                        username:
                          type: string
                        password:
                          type: string
                        database:
                          type: string
          status:
            type: object
            properties:
              phase:
                type: string
                description: "Application phase"
                enum: [Pending, Installing, Ready, Updating, Deleting, Failed]
              message:
                type: string
                description: "Status message"
              components:
                type: array
                description: "Component statuses"
                items:
                  type: object
                  properties:
                    name:
                      type: string
                    phase:
                      type: string
                      enum: [Pending, Creating, Ready, Updating, Deleting, Failed]
                    message:
                      type: string
              conditions:
                type: array
                description: "Application conditions"
                items:
                  type: object
                  properties:
                    type:
                      type: string
                    status:
                      type: string
                    reason:
                      type: string
                    message:
                      type: string
                    lastTransitionTime:
                      type: string
                      format: date-time
              installedAt:
                type: string
                format: date-time
              updatedAt:
                type: string
                format: date-time
    additionalPrinterColumns:
    - name: Phase
      type: string
      jsonPath: .status.phase
    - name: Components
      type: integer
      jsonPath: .status.components
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp
  scope: Namespaced
  names:
    plural: applications
    singular: application
    kind: Application
    shortNames:
    - app
    categories:
    - all
    - example
```

## Troubleshooting

### Common Issues

1. **CRD not being recognized**
   - Check CRD creation status
   - Verify API group and version
   - Review CRD schema validation
   - Check Kubernetes version compatibility

2. **Custom resource validation failing**
   - Review OpenAPI schema
   - Check required fields
   - Verify data types and patterns
   - Examine error messages

3. **Controller not reconciling**
   - Check controller logs
   - Verify RBAC permissions
   - Review event logs
   - Check network connectivity

### Debug Commands

```bash
# Check CRD status
kubectl get crd
kubectl describe crd <crd-name>

# Check custom resources
kubectl get <cr-kind>
kubectl describe <cr-kind> <cr-name>

# Check CRD validation
kubectl apply -f custom-resource.yaml --dry-run=client

# Check controller logs
kubectl logs -f deployment/<controller-name> -n <namespace>

# Check events
kubectl get events --field-selector involvedObject.kind=<cr-kind>
```

## Advanced Topics

### 1. CRD Conversion Webhooks

```yaml
# Conversion webhook configuration
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: conversions.example.com
spec:
  group: example.com
  versions:
  - name: v1alpha1
    served: true
    storage: false
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              oldField:
                type: string
  - name: v1beta1
    served: true
    storage: false
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              newField:
                type: string
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              finalField:
                type: string
  conversion:
    strategy: Webhook
    webhook:
      conversionReviewVersions: ["v1", "v1beta1"]
      clientConfig:
        service:
          name: conversion-webhook
          namespace: example-system
          path: "/convert"
        caBundle: <base64-encoded-ca-cert>
```

### 2. CRD with Finalizers

```yaml
# CRD with finalizer support
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: finalizers.example.com
spec:
  group: example.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              name:
                type: string
          status:
            type: object
            properties:
              phase:
                type: string
              finalizers:
                type: array
                items:
                  type: string
  scope: Namespaced
  names:
    plural: finalizers
    singular: finalizer
    kind: Finalizer
```

### 3. CRD with Webhook Validation

```yaml
# CRD with admission webhook
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: validating-webhook-configuration
webhooks:
- name: validate.example.com
  rules:
  - apiGroups: ["example.com"]
    apiVersions: ["v1"]
    operations: ["CREATE", "UPDATE"]
    resources: ["databases"]
  admissionReviewVersions: ["v1", "v1beta1"]
  clientConfig:
    service:
      name: validation-webhook
      namespace: example-system
      path: "/validate"
    caBundle: <base64-encoded-ca-cert>
  sideEffects: None
  timeoutSeconds: 5
```

## Conclusion

Custom Resource Definitions are a fundamental building block for extending Kubernetes functionality. They provide a powerful mechanism for creating domain-specific APIs and objects that integrate seamlessly with the Kubernetes ecosystem.

Understanding CRDs, their design patterns, and best practices is essential for building sophisticated Kubernetes operators and controllers. As the Kubernetes ecosystem continues to evolve, CRDs remain a critical component for platform engineering and custom application management.

By following the design principles and implementation patterns outlined in this guide, developers can create robust, maintainable, and scalable CRDs that meet the needs of complex enterprise applications while maintaining the declarative nature and operational excellence that Kubernetes provides.