# Ingest session

You are maintaining the wiki described in `CLAUDE.md`. Read that file first if
you have not already in this session.

Ingest the units below **in order**. For each one: identify the components,
procedures and errors it describes; check the related index entries for pages
that already cover them; update those pages and create only what is genuinely
missing; add cross-references in both directions; update `index.md`; append one
entry to `log.md`.

Do not read other files under `raw/` unless a unit is unusable on its own - if
that happens, say so rather than guessing. When you are done, list the unit ids
you completed so they can be marked off:

    rhkb done <unit-ids>

## Related index entries

- [[gpu-operator-clusterpolicy]] (component) - the ClusterPolicy CRD that drives the NVIDIA GPU Operator; indexes its managed components [gpu-operator]
- [[nvidia-dcgm]] (component) - DCGM hostengine pod deployed by the GPU Operator [gpu-operator]
- [[nvidia-driver]] (component) - GPU kernel driver deployed by the GPU Operator [gpu-operator]
- [[nvidia-cc-manager]] (component) - Confidential Computing mode manager for compatible GPUs [gpu-operator]
- [[nvidia-gdrcopy]] (component) - GDRCopy driver for direct GPU/CPU memory copies [gpu-operator]
- [[nvidia-gpudirect-storage]] (component) - GPUDirect Storage, experimental per upstream schema [gpu-operator]
- [[nvidia-mig-manager]] (component) - MIG strategy and MIG Manager for GPU partitioning [gpu-operator]

(7 of 12 pages shown - open only the ones you need.)

## Units

### Ingest unit `gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md:nvidiacom-clusterpoliciesyaml#5`
- source: gpu-operator (NVIDIA GPU Operator)
- document: gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md
- section: nvidia.com_clusterpolicies.yaml (5/7)
- url: https://github.com/NVIDIA/gpu-operator/blob/main/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml

---

pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Limits describes the maximum amount of compute resources allowed.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                      requests:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Requests describes the minimum amount of compute resources required.
                          If Requests is omitted for a container, it defaults to Limits if that is explicitly specified,
                          otherwise to an implementation-defined value. Requests cannot exceed Limits.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                    type: object
                  version:
                    description: NVIDIA MIG Manager image tag
                    type: string
                type: object
              nodeStatusExporter:
                description: NodeStatusExporter spec
                properties:
                  args:
                    description: 'Optional: List of arguments'
                    items:
                      type: string
                    type: array
                  enabled:
                    description: Enabled indicates if deployment of Node Status Exporter
                      is enabled.
                    type: boolean
                  env:
                    description: 'Optional: List of environment variables'
                    items:
                      description: EnvVar represents an environment variable present
                        in a Container.
                      properties:
                        name:
                          description: Name of the environment variable.
                          type: string
                        value:
                          description: Value of the environment variable.
                          type: string
                      required:
                      - name
                      type: object
                    type: array
                  hostNetwork:
                    description: HostNetwork indicates whether the Node Status Exporter
                      pod uses the host's network namespace.
                    type: boolean
                  image:
                    description: Node Status Exporter image name
                    pattern: '[a-zA-Z0-9\-]+'
                    type: string
                  imagePullPolicy:
                    description: Image pull policy
                    type: string
                  imagePullSecrets:
                    description: Image pull secrets
                    items:
                      type: string
                    type: array
                  repository:
                    description: Node Status Exporterimage repository
                    type: string
                  resources:
                    description: 'Optional: Define resources requests and limits for
                      each pod'
                    properties:
                      limits:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Limits describes the maximum amount of compute resources allowed.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                      requests:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Requests describes the minimum amount of compute resources required.
                          If Requests is omitted for a container, it defaults to Limits if that is explicitly specified,
                          otherwise to an implementation-defined value. Requests cannot exceed Limits.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                    type: object
                  version:
                    description: Node Status Exporterimage tag
                    type: string
                type: object
              operator:
                description: Operator component spec
                properties:
                  annotations:
                    additionalProperties:
                      type: string
                    description: |-
                      Optional: Annotations is an unstructured key value map stored with a resource that may be
                      set by external tools to store and retrieve arbitrary metadata. They are not
                      queryable and should be preserved when modifying objects.
                    type: object
                  defaultRuntime:
                    description: 'Deprecated: DefaultRuntime is no longer used by
                      the gpu-operator. This is instead, detected at runtime.'
                    type: string
                  initContainer:
                    description: 'Deprecated: InitContainerSpec describes configuration
                      for initContainer image used with all components'
                    properties:
                      image:
                        description: Image represents image name
                        pattern: '[a-zA-Z0-9\-]+'
                        type: string
                      imagePullPolicy:
                        description: Image pull policy
                        type: string
                      imagePullSecrets:
                        description: Image pull secrets
                        items:
                          type: string
                        type: array
                      repository:
                        description: Repository represents image repository path
                        type: string
                      version:
                        description: Version represents image tag(version)
                        type: string
                    type: object
                  labels:
                    additionalProperties:
                      type: string
                    description: |-
                      Optional: Map of string keys and values that can be used to organize and categorize
                      (scope and select) objects. May match selectors of replication controllers
                      and services.
                    type: object
                  metrics:
                    description: Metrics configuration for NVIDIA GPU Operator
                    properties:
                      serviceMonitor:
                        description: 'Optional: ServiceMonitor configuration for NVIDIA
                          GPU Operator'
                        properties:
                          additionalLabels:
                            additionalProperties:
                              type: string
                            description: AdditionalLabels to add to ServiceMonitor
                              instance
                            type: object
                          enabled:
                            description: Enabled indicates if ServiceMonitor is deployed
                            type: boolean
                          honorLabels:
                            description: HonorLabels chooses the metric’s labels on
                              collisions with target labels.
                            type: boolean
                          interval:
                            description: |-
                              Interval at which metrics should be scraped. If not specified, Prometheus’ global scrape interval is used.
                              Supported units: y, w, d, h, m, s, ms
                            pattern: ^(0|(([0-9]+)y)?(([0-9]+)w)?(([0-9]+)d)?(([0-9]+)h)?(([0-9]+)m)?(([0-9]+)s)?(([0-9]+)ms)?)$
                            type: string
                          relabelings:
                            description: Relabelings allows to rewrite labels on metric
                              sets
                            items:
                              description: |-
                                RelabelConfig allows dynamic rewriting of the label set for targets, alerts,
                                scraped samples and remote write samples.

                                More info: https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config
                              properties:
                                action:
                                  default: replace
                                  description: |-
                                    action to perform based on the regex matching.

                                    `Uppercase` and `Lowercase` actions require Prometheus >= v2.36.0.
                                    `DropEqual` and `KeepEqual` actions require Prometheus >= v2.41.0.

                                    Default: "Replace"
                                  enum:
                                  - replace
                                  - Replace
                                  - keep
                                  - Keep
                                  - drop
                                  - Drop
                                  - hashmod
                                  - HashMod
                                  - labelmap
                                  - LabelMap
                                  - labeldrop
                                  - LabelDrop
                                  - labelkeep
                                  - LabelKeep
                                  - lowercase
                                  - Lowercase
                                  - uppercase
                                  - Uppercase
                                  - keepequal
                                  - KeepEqual
                                  - dropequal
                                  - DropEqual
                                  type: string
                                modulus:
                                  description: |-
                                    modulus to take of the hash of the source label values.

                                    Only applicable when the action is `HashMod`.
                                  format: int64
                                  minimum: 0
                                  type: integer
                                regex:
                                  description: regex defines the regular expression
                                    against which the extracted value is matched.
                                  type: string
                                replacement:
                                  description: |-
                                    replacement value against which a Replace action is performed if the
                                    regular expression matches.

                                    Regex capture groups are available.
                                  type: string
                                separator:
                                  description: separator defines the string between
                                    concatenated SourceLabels.
                                  type: string
                                sourceLabels:
                                  description: |-
                                    sourceLabels defines the source labels select values from existing labels. Their content is
                                    concatenated using the configured Separator and matched against the
                                    configured regular expression.
                                  items:
                                    description: |-
                                      LabelName is a valid Prometheus label name.
                                      For Prometheus 3.x, a label name is valid if it contains UTF-8 characters.
                                      For Prometheus 2.x, a label name is only valid if it contains ASCII characters, letters, numbers, as well as underscores.
                                    type: string
                                  type: array
                                targetLabel:
                                  description: |-
                                    targetLabel defines the label to which the resulting string is written in a replacement.

                                    It is mandatory for `Replace`, `HashMod`, `Lowercase`, `Uppercase`,
                                    `KeepEqual` and `DropEqual` actions.

                                    Regex capture groups are available.
                                  type: string
                              type: object
                            type: array
                          scrapeTimeout:
                            description: |-
                              ScrapeTimeout to use when scraping metrics. Must not be greater than Interval.
                              If not specified, Prometheus' global scrape timeout is used.
                              Supported units: y, w, d, h, m, s, ms
                            pattern: ^(0|(([0-9]+)y)?(([0-9]+)w)?(([0-9]+)d)?(([0-9]+)h)?(([0-9]+)m)?(([0-9]+)s)?(([0-9]+)ms)?)$
                            type: string
                        type: object
                    type: object
                  runtimeClass:
                    default: nvidia
                    type: string
                  use_ocp_driver_toolkit:
                    description: UseOpenShiftDriverToolkit indicates if DriverToolkit
                      image should be used on OpenShift to build and install driver
                      modules
                    type: boolean
                type: object
              psa:
                description: PSA defines spec for PodSecurityAdmission configuration
                properties:
                  enabled:
                    description: Enabled indicates if PodSecurityAdmission configuration
                      needs to be enabled for all Pods
                    type: boolean
                type: object
              psp:
                description: |-
                  Deprecated: Pod Security Policies are no longer supported. Please use PodSecurityAdmission instead
                  PSP defines spec for handling PodSecurityPolicies
                properties:
                  enabled:
                    description: Enabled indicates if PodSecurityPolicies needs to
                      be enabled for all Pods
                    type: boolean
                type: object
              sandboxDevicePlugin:
                description: SandboxDevicePlugin component spec
                properties:
                  args:
                    description: 'Optional: List of arguments'
                    items:
                      type: string
                    type: array
                  enabled:
                    description: Enabled indicates if deployment of NVIDIA Sandbox
                      Device Plugin through operator is enabled
                    type: boolean
                  env:
                    description: 'Optional: List of environment variables'
                    items:
                      description: EnvVar represents an environment variable present
                        in a Container.
                      properties:
                        name:
                          description: Name of the environment variable.
                          type: string
                        value:
                          description: Value of the environment variable.
                          type: string
                      required:
                      - name
                      type: object
                    type: array
                  hostNetwork:
                    description: HostNetwork indicates whether the Sandbox Device
                      Plugin pod uses the host's network namespace.
                    type: boolean
                  image:
                    description: NVIDIA Sandbox Device Plugin image name
                    pattern: '[a-zA-Z0-9\-]+'
                    type: string
                  imagePullPolicy:
                    description: Image pull policy
                    type: string
                  imagePullSecrets:
                    description: Image pull secrets
                    items:
                      type: string
                    type: array
                  repository:
                    description: NVIDIA Sandbox Device Plugin image repository
                    type: string
                  resources:
                    description: 'Optional: Define resources requests and limits for
                      each pod'
                    properties:
                      limits:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Limits describes the maximum amount of compute resources allowed.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                      requests:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Requests describes the minimum amount of compute resources required.
                          If Requests is omitted for a container, it defaults to Limits if that is explicitly specified,
                          otherwise to an implementation-defined value. Requests cannot exceed Limits.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                    type: object
                  version:
                    description: NVIDIA Sandbox Device Plugin image tag
                    type: string
                type: object
              sandboxWorkloads:
                description: SandboxWorkloads defines the spec for handling sandbox
                  workloads (i.e. Virtual Machines)
                properties:
                  defaultWorkload:
                    default: container
                    description: |-
                      DefaultWorkload indicates the default GPU workload type to configure
                      worker nodes in the cluster for
                    enum:
                    - container
                    - vm-passthrough
                    - vm-vgpu
                    type: string
                  enabled:
                    description: |-
                      Enabled indicates if the GPU Operator should manage additional operands required
                      for sandbox workloads (i.e. VFIO Manager, vGPU Manager, and additional device plugins)
                    type: boolean
                  mode:
                    default: kubevirt
                    description: |-
                      Mode indicates the sandbox mode. Accepted values are "kubevirt"
                      and "kata". The default value is "kubevirt".
                    enum:
                    - kubevirt
                    - kata
                    type: string
                type: object
              toolkit:
                description: Toolkit component spec
                properties:
                  args:
                    description: 'Optional: List of arguments'
                    items:
                      type: string
                    type: array
                  enabled:
                    description: Enabled indicates if deployment of NVIDIA Container
                      Toolkit through operator is enabled
                    type: boolean
                  env:
                    description: 'Optional: List of environment variables'
                    items:
                      description: EnvVar represents an environment variable present
                        in a Container.
                      properties:
                        name:
                          description: Name of the environment variable.
                          type: string
                        value:
                          description: Value of the environment variable.
                          type: string
                      required:
                      - name
                      type: object
                    type: array
                  hostNetwork:
                    description: HostNetwork indicates whether the Container Toolkit
                      pod uses the host's network namespace.
                    type: boolean
                  image:
                    description: NVIDIA Container Toolkit image name
                    pattern: '[a-zA-Z0-9\-]+'
                    type: string
                  imagePullPolicy:
                    description: Image pull policy
                    type: string
                  imagePullSecrets:
                    description: Image pull secrets
                    items:
                      type: string
                    type: array
                  installDir:
                    default: /usr/local/nvidia
                    description: Toolkit install directory on the host
                    type: string
                  repository:
                    description: NVIDIA Container Toolkit image repository
                    type: string
                  resources:
                    description: 'Optional: Define resources requests and limits for
                      each pod'
                    properties:
                      limits:
                        additionalProperties:
                          anyOf:
                          - type: integer

### Ingest unit `gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md:nvidiacom-clusterpoliciesyaml#6`
- source: gpu-operator (NVIDIA GPU Operator)
- document: gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md
- section: nvidia.com_clusterpolicies.yaml (6/7)
- url: https://github.com/NVIDIA/gpu-operator/blob/main/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml

---

- type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Limits describes the maximum amount of compute resources allowed.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                      requests:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Requests describes the minimum amount of compute resources required.
                          If Requests is omitted for a container, it defaults to Limits if that is explicitly specified,
                          otherwise to an implementation-defined value. Requests cannot exceed Limits.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                    type: object
                  version:
                    description: NVIDIA Container Toolkit image tag
                    type: string
                type: object
              validator:
                description: Validator defines the spec for operator-validator daemonset
                properties:
                  args:
                    description: 'Optional: List of arguments'
                    items:
                      type: string
                    type: array
                  cuda:
                    description: CUDA validator spec
                    properties:
                      env:
                        description: 'Optional: List of environment variables'
                        items:
                          description: EnvVar represents an environment variable present
                            in a Container.
                          properties:
                            name:
                              description: Name of the environment variable.
                              type: string
                            value:
                              description: Value of the environment variable.
                              type: string
                          required:
                          - name
                          type: object
                        type: array
                    type: object
                  driver:
                    description: Toolkit validator spec
                    properties:
                      env:
                        description: 'Optional: List of environment variables'
                        items:
                          description: EnvVar represents an environment variable present
                            in a Container.
                          properties:
                            name:
                              description: Name of the environment variable.
                              type: string
                            value:
                              description: Value of the environment variable.
                              type: string
                          required:
                          - name
                          type: object
                        type: array
                    type: object
                  env:
                    description: 'Optional: List of environment variables'
                    items:
                      description: EnvVar represents an environment variable present
                        in a Container.
                      properties:
                        name:
                          description: Name of the environment variable.
                          type: string
                        value:
                          description: Value of the environment variable.
                          type: string
                      required:
                      - name
                      type: object
                    type: array
                  hostNetwork:
                    description: HostNetwork indicates whether the Validator pod uses
                      the host's network namespace.
                    type: boolean
                  image:
                    description: Validator image name
                    pattern: '[a-zA-Z0-9\-]+'
                    type: string
                  imagePullPolicy:
                    description: Image pull policy
                    type: string
                  imagePullSecrets:
                    description: Image pull secrets
                    items:
                      type: string
                    type: array
                  plugin:
                    description: Plugin validator spec
                    properties:
                      env:
                        description: 'Optional: List of environment variables'
                        items:
                          description: EnvVar represents an environment variable present
                            in a Container.
                          properties:
                            name:
                              description: Name of the environment variable.
                              type: string
                            value:
                              description: Value of the environment variable.
                              type: string
                          required:
                          - name
                          type: object
                        type: array
                    type: object
                  repository:
                    description: Validator image repository
                    type: string
                  resources:
                    description: 'Optional: Define resources requests and limits for
                      each pod'
                    properties:
                      limits:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Limits describes the maximum amount of compute resources allowed.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                      requests:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Requests describes the minimum amount of compute resources required.
                          If Requests is omitted for a container, it defaults to Limits if that is explicitly specified,
                          otherwise to an implementation-defined value. Requests cannot exceed Limits.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                    type: object
                  toolkit:
                    description: Toolkit validator spec
                    properties:
                      env:
                        description: 'Optional: List of environment variables'
                        items:
                          description: EnvVar represents an environment variable present
                            in a Container.
                          properties:
                            name:
                              description: Name of the environment variable.
                              type: string
                            value:
                              description: Value of the environment variable.
                              type: string
                          required:
                          - name
                          type: object
                        type: array
                    type: object
                  version:
                    description: Validator image tag
                    type: string
                  vfioPCI:
                    description: VfioPCI validator spec
                    properties:
                      env:
                        description: 'Optional: List of environment variables'
                        items:
                          description: EnvVar represents an environment variable present
                            in a Container.
                          properties:
                            name:
                              description: Name of the environment variable.
                              type: string
                            value:
                              description: Value of the environment variable.
                              type: string
                          required:
                          - name
                          type: object
                        type: array
                    type: object
                  vgpuDevices:
                    description: VGPUDevices validator spec
                    properties:
                      env:
                        description: 'Optional: List of environment variables'
                        items:
                          description: EnvVar represents an environment variable present
                            in a Container.
                          properties:
                            name:
                              description: Name of the environment variable.
                              type: string
                            value:
                              description: Value of the environment variable.
                              type: string
                          required:
                          - name
                          type: object
                        type: array
                    type: object
                  vgpuManager:
                    description: VGPUManager validator spec
                    properties:
                      env:
                        description: 'Optional: List of environment variables'
                        items:
                          description: EnvVar represents an environment variable present
                            in a Container.
                          properties:
                            name:
                              description: Name of the environment variable.
                              type: string
                            value:
                              description: Value of the environment variable.
                              type: string
                          required:
                          - name
                          type: object
                        type: array
                    type: object
                type: object
              vfioManager:
                description: VFIOManager for configuration to deploy VFIO-PCI Manager
                properties:
                  args:
                    description: 'Optional: List of arguments'
                    items:
                      type: string
                    type: array
                  driverManager:
                    description: DriverManager represents configuration for NVIDIA
                      Driver Manager
                    properties:
                      env:
                        description: 'Optional: List of environment variables'
                        items:
                          description: EnvVar represents an environment variable present
                            in a Container.
                          properties:
                            name:
                              description: Name of the environment variable.
                              type: string
                            value:
                              description: Value of the environment variable.
                              type: string
                          required:
                          - name
                          type: object
                        type: array
                      image:
                        description: Image represents NVIDIA Driver Manager image
                          name
                        pattern: '[a-zA-Z0-9\-]+'
                        type: string
                      imagePullPolicy:
                        description: Image pull policy
                        type: string
                      imagePullSecrets:
                        description: Image pull secrets
                        items:
                          type: string
                        type: array
                      repository:
                        description: Repository represents Driver Managerrepository
                          path
                        type: string
                      version:
                        description: Version represents NVIDIA Driver Manager image
                          tag(version)
                        type: string
                    type: object
                  enabled:
                    description: Enabled indicates if deployment of VFIO Manager is
                      enabled
                    type: boolean
                  env:
                    description: 'Optional: List of environment variables'
                    items:
                      description: EnvVar represents an environment variable present
                        in a Container.
                      properties:
                        name:
                          description: Name of the environment variable.
                          type: string
                        value:
                          description: Value of the environment variable.
                          type: string
                      required:
                      - name
                      type: object
                    type: array
                  hostNetwork:
                    description: HostNetwork indicates whether the VFIO Manager pod
                      uses the host's network namespace.
                    type: boolean
                  image:
                    description: VFIO Manager image name
                    pattern: '[a-zA-Z0-9\-]+'
                    type: string
                  imagePullPolicy:
                    description: Image pull policy
                    type: string
                  imagePullSecrets:
                    description: Image pull secrets
                    items:
                      type: string
                    type: array
                  repository:
                    description: VFIO Manager image repository
                    type: string
                  resources:
                    description: 'Optional: Define resources requests and limits for
                      each pod'
                    properties:
                      limits:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Limits describes the maximum amount of compute resources allowed.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                      requests:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Requests describes the minimum amount of compute resources required.
                          If Requests is omitted for a container, it defaults to Limits if that is explicitly specified,
                          otherwise to an implementation-defined value. Requests cannot exceed Limits.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                    type: object
                  version:
                    description: VFIO Manager image tag
                    type: string
                type: object
              vgpuDeviceManager:
                description: VGPUDeviceManager spec
                properties:
                  args:
                    description: 'Optional: List of arguments'
                    items:
                      type: string
                    type: array
                  config:
                    description: NVIDIA vGPU devices configuration for NVIDIA vGPU
                      Device Manager container
                    properties:
                      default:
                        default: default
                        description: Default config name within the ConfigMap
                        type: string
                      name:
                        description: ConfigMap name
                        type: string
                    type: object
                  enabled:
                    description: Enabled indicates if deployment of NVIDIA vGPU Device
                      Manager is enabled
                    type: boolean
                  env:
                    description: 'Optional: List of environment variables'
                    items:
                      description: EnvVar represents an environment variable present
                        in a Container.
                      properties:
                        name:
                          description: Name of the environment variable.
                          type: string
                        value:
                          description: Value of the environment variable.
                          type: string
                      required:
                      - name
                      type: object
                    type: array
                  hostNetwork:
                    description: HostNetwork indicates whether the vGPU Device Manager
                      pod uses the host's network namespace.
                    type: boolean
                  image:
                    description: NVIDIA vGPU Device Manager image name
                    pattern: '[a-zA-Z0-9\-]+'
                    type: string
                  imagePullPolicy:
                    description: Image pull policy
                    type: string
                  imagePullSecrets:
                    description: Image pull secrets
                    items:
                      type: string
                    type: array
                  repository:
                    description: NVIDIA vGPU Device Manager image repository
                    type: string
                  resources:
                    description: 'Optional: Define resources requests and limits for
                      each pod'
                    properties:
                      limits:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Limits describes the maximum amount of compute resources allowed.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                      requests:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Requests describes the minimum amount of compute resources required.
                          If Requests is omitted for a container, it defaults to Limits if that is explicitly specified,
                          otherwise to an implementation-defined value. Requests cannot exceed Limits.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                    type: object
                  version:
                    description: NVIDIA vGPU Device Manager image tag
                    type: string
                type: object
              vgpuManager:
                description: VGPUManager component spec
                properties:
                  args:
                    description: 'Optional: List of arguments'
                    items:
                      type: string
                    type: array
                  driverManager:
                    description: DriverManager represents configuration for NVIDIA
                      Driver Manager initContainer
                    properties:
                      env:
                        description: 'Optional: List of environment variables'
                        items:
                          description: EnvVar represents an environment variable present
                            in a Container.
                          properties:
                            name:
                              description: Name of the environment variable.
                              type: string
                            value:
                              description: Value of the environment variable.
                              type: string
                          required:
                          - name
                          type: object
                        type: array
                      image:
                        description: Image represents NVIDIA Driver Manager image
                          name
                        pattern: '[a-zA-Z0-9\-]+'
                        type: string
                      imagePullPolicy:
                        description: Image pull policy
                        type: string
                      imagePullSecrets:
                        description: Image pull secrets
                        items:
                          type: string
                        type: array
                      repository:

### Ingest unit `gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md:nvidiacom-clusterpoliciesyaml#7`
- source: gpu-operator (NVIDIA GPU Operator)
- document: gpu-operator/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml.md
- section: nvidia.com_clusterpolicies.yaml (7/7)
- url: https://github.com/NVIDIA/gpu-operator/blob/main/deployments/gpu-operator/crds/nvidia.com_clusterpolicies.yaml

---

description: Repository represents Driver Managerrepository
                          path
                        type: string
                      version:
                        description: Version represents NVIDIA Driver Manager image
                          tag(version)
                        type: string
                    type: object
                  enabled:
                    description: Enabled indicates if deployment of NVIDIA vGPU Manager
                      through operator is enabled
                    type: boolean
                  env:
                    description: 'Optional: List of environment variables'
                    items:
                      description: EnvVar represents an environment variable present
                        in a Container.
                      properties:
                        name:
                          description: Name of the environment variable.
                          type: string
                        value:
                          description: Value of the environment variable.
                          type: string
                      required:
                      - name
                      type: object
                    type: array
                  hostNetwork:
                    description: HostNetwork indicates whether the vGPU Manager pod
                      uses the host's network namespace.
                    type: boolean
                  image:
                    description: NVIDIA vGPU Manager  image name
                    pattern: '[a-zA-Z0-9\-]+'
                    type: string
                  imagePullPolicy:
                    description: Image pull policy
                    type: string
                  imagePullSecrets:
                    description: Image pull secrets
                    items:
                      type: string
                    type: array
                  kernelModuleConfig:
                    description: 'Optional: Kernel module configuration parameters
                      for the vGPU manager'
                    properties:
                      name:
                        type: string
                    type: object
                  repository:
                    description: NVIDIA vGPU Manager image repository
                    type: string
                  resources:
                    description: 'Optional: Define resources requests and limits for
                      each pod'
                    properties:
                      limits:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Limits describes the maximum amount of compute resources allowed.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                      requests:
                        additionalProperties:
                          anyOf:
                          - type: integer
                          - type: string
                          pattern: ^(\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))(([KMGTPE]i)|[numkMGTPE]|([eE](\+|-)?(([0-9]+(\.[0-9]*)?)|(\.[0-9]+))))?$
                          x-kubernetes-int-or-string: true
                        description: |-
                          Requests describes the minimum amount of compute resources required.
                          If Requests is omitted for a container, it defaults to Limits if that is explicitly specified,
                          otherwise to an implementation-defined value. Requests cannot exceed Limits.
                          More info: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/
                        type: object
                    type: object
                  version:
                    description: NVIDIA vGPU Manager image tag
                    type: string
                type: object
            required:
            - daemonsets
            - dcgm
            - dcgmExporter
            - devicePlugin
            - driver
            - gfd
            - nodeStatusExporter
            - operator
            - toolkit
            type: object
          status:
            description: ClusterPolicyStatus defines the observed state of ClusterPolicy
            properties:
              conditions:
                description: Conditions is a list of conditions representing the ClusterPolicy's
                  current state.
                items:
                  description: Condition contains details for one aspect of the current
                    state of this API Resource.
                  properties:
                    lastTransitionTime:
                      description: |-
                        lastTransitionTime is the last time the condition transitioned from one status to another.
                        This should be when the underlying condition changed.  If that is not known, then using the time when the API field changed is acceptable.
                      format: date-time
                      type: string
                    message:
                      description: |-
                        message is a human readable message indicating details about the transition.
                        This may be an empty string.
                      maxLength: 32768
                      type: string
                    observedGeneration:
                      description: |-
                        observedGeneration represents the .metadata.generation that the condition was set based upon.
                        For instance, if .metadata.generation is currently 12, but the .status.conditions[x].observedGeneration is 9, the condition is out of date
                        with respect to the current state of the instance.
                      format: int64
                      minimum: 0
                      type: integer
                    reason:
                      description: |-
                        reason contains a programmatic identifier indicating the reason for the condition's last transition.
                        Producers of specific condition types may define expected values and meanings for this field,
                        and whether the values are considered a guaranteed API.
                        The value should be a CamelCase string.
                        This field may not be empty.
                      maxLength: 1024
                      minLength: 1
                      pattern: ^[A-Za-z]([A-Za-z0-9_,:]*[A-Za-z0-9_])?$
                      type: string
                    status:
                      description: status of the condition, one of True, False, Unknown.
                      enum:
                      - "True"
                      - "False"
                      - Unknown
                      type: string
                    type:
                      description: type of condition in CamelCase or in foo.example.com/CamelCase.
                      maxLength: 316
                      pattern: ^([a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)*/)?(([A-Za-z0-9][-A-Za-z0-9_.]*)?[A-Za-z0-9])$
                      type: string
                  required:
                  - lastTransitionTime
                  - message
                  - reason
                  - status
                  - type
                  type: object
                type: array
              namespace:
                description: Namespace indicates a namespace in which the operator
                  is installed
                type: string
              state:
                description: State indicates status of ClusterPolicy
                enum:
                - ignored
                - ready
                - notReady
                type: string
            required:
            - state
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
```

### Ingest unit `gpu-operator/deployments/gpu-operator/crds/nvidia.com_gpuclusters.yaml.md:nvidiacom-gpuclustersyaml#1`
- source: gpu-operator (NVIDIA GPU Operator)
- document: gpu-operator/deployments/gpu-operator/crds/nvidia.com_gpuclusters.yaml.md
- section: nvidia.com_gpuclusters.yaml (1/3)
- url: https://github.com/NVIDIA/gpu-operator/blob/main/deployments/gpu-operator/crds/nvidia.com_gpuclusters.yaml

---

```yaml
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.21.0
  name: gpuclusters.nvidia.com
spec:
  group: nvidia.com
  names:
    kind: GPUCluster
    listKind: GPUClusterList
    plural: gpuclusters
    shortNames:
    - gc
    singular: gpucluster
  scope: Cluster
  versions:
  - additionalPrinterColumns:
    - jsonPath: .status.state
      name: Status
      type: string
    - jsonPath: .metadata.creationTimestamp
      name: Age
      type: date
    name: v1alpha1
    schema:
      openAPIV3Schema:
        description: GPUCluster is the Schema for the gpuclusters API
        properties:
          apiVersion:
            description: |-
              APIVersion defines the versioned schema of this representation of an object.
              Servers should convert recognized schemas to the latest internal value, and
              may reject unrecognized values.
              More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources
            type: string
          kind:
            description: |-
              Kind is a string value representing the REST resource this object represents.
              Servers may infer this from the endpoint the client submits requests to.
              Cannot be updated.
              In CamelCase.
              More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds
            type: string
          metadata:
            type: object
          spec:
            description: |-
              GPUClusterSpec defines the desired state of GPUCluster, the DRA-based
              software-enablement stack. Unlike ClusterPolicy, it does not manage the NVIDIA driver
              or the device-plugin; the driver is installed separately (host-installed or via an
              NVIDIADriver CR) and GPUCluster waits for driver readiness before proceeding.
            properties:
              daemonsets:
                description: |-
                  Daemonsets defines the common configuration applied to all DaemonSets deployed
                  by the GPUCluster controller.
                properties:
                  annotations:
                    additionalProperties:
                      type: string
                    description: |-
                      Optional: Annotations is an unstructured key value map stored with a resource that may be
                      set by external tools to store and retrieve arbitrary metadata. They are not
                      queryable and should be preserved when modifying objects.
                    type: object
                  labels:
                    additionalProperties:
                      type: string
                    description: |-
                      Optional: Map of string keys and values that can be used to organize and categorize
                      (scope and select) objects. May match selectors of replication controllers
                      and services.
                    type: object
                  podSecurityContext:
                    description: 'Optional: Set pod-level security context for all
                      DaemonSet pods (applies as defaults to all containers)'
                    properties:
                      appArmorProfile:
                        description: |-
                          appArmorProfile is the AppArmor options to use by the containers in this pod.
                          Note that this field cannot be set when spec.os.name is windows.
                        properties:
                          localhostProfile:
                            description: |-
                              localhostProfile indicates a profile loaded on the node that should be used.
                              The profile must be preconfigured on the node to work.
                              Must match the loaded name of the profile.
                              Must be set if and only if type is "Localhost".
                            type: string
                          type:
                            description: |-
                              type indicates which kind of AppArmor profile will be applied.
                              Valid options are:
                                Localhost - a profile pre-loaded on the node.
                                RuntimeDefault - the container runtime's default profile.
                                Unconfined - no AppArmor enforcement.
                            type: string
                        required:
                        - type
                        type: object
                      fsGroup:
                        description: |-
                          A special supplemental group that applies to all containers in a pod.
                          Some volume types allow the Kubelet to change the ownership of that volume
                          to be owned by the pod:

                          1. The owning GID will be the FSGroup
                          2. The setgid bit is set (new files created in the volume will be owned by FSGroup)
                          3. The permission bits are OR'd with rw-rw----

                          If unset, the Kubelet will not modify the ownership and permissions of any volume.
                          Note that this field cannot be set when spec.os.name is windows.
                        format: int64
                        type: integer
                      fsGroupChangePolicy:
                        description: |-
                          fsGroupChangePolicy defines behavior of changing ownership and permission of the volume
                          before being exposed inside Pod. This field will only apply to
                          volume types which support fsGroup based ownership(and permissions).
                          It will have no effect on ephemeral volume types such as: secret, configmaps
                          and emptydir.
                          Valid values are "OnRootMismatch" and "Always". If not specified, "Always" is used.
                          Note that this field cannot be set when spec.os.name is windows.
                        type: string
                      runAsGroup:
                        description: |-
                          The GID to run the entrypoint of the container process.
                          Uses runtime default if unset.
                          May also be set in SecurityContext.  If set in both SecurityContext and
                          PodSecurityContext, the value specified in SecurityContext takes precedence
                          for that container.
                          Note that this field cannot be set when spec.os.name is windows.
                        format: int64
                        type: integer
                      runAsNonRoot:
                        description: |-
                          Indicates that the container must run as a non-root user.
                          If true, the Kubelet will validate the image at runtime to ensure that it
                          does not run as UID 0 (root) and fail to start the container if it does.
                          If unset or false, no such validation will be performed.
                          May also be set in SecurityContext.  If set in both SecurityContext and
                          PodSecurityContext, the value specified in SecurityContext takes precedence.
                        type: boolean
                      runAsUser:
                        description: |-
                          The UID to run the entrypoint of the container process.
                          Defaults to user specified in image metadata if unspecified.
                          May also be set in SecurityContext.  If set in both SecurityContext and
                          PodSecurityContext, the value specified in SecurityContext takes precedence
                          for that container.
                          Note that this field cannot be set when spec.os.name is windows.
                        format: int64
                        type: integer
                      seLinuxChangePolicy:
                        description: |-
                          seLinuxChangePolicy defines how the container's SELinux label is applied to all volumes used by the Pod.
                          It has no effect on nodes that do not support SELinux or to volumes does not support SELinux.
                          Valid values are "MountOption" and "Recursive".

                          "Recursive" means relabeling of all files on all Pod volumes by the container runtime.
                          This may be slow for large volumes, but allows mixing privileged and unprivileged Pods sharing the same volume on the same node.

                          "MountOption" mounts all eligible Pod volumes with `-o context` mount option.
                          This requires all Pods that share the same volume to use the same SELinux label.
                          It is not possible to share the same volume among privileged and unprivileged Pods.
                          Eligible volumes are in-tree FibreChannel and iSCSI volumes, and all CSI volumes
                          whose CSI driver announces SELinux support by setting spec.seLinuxMount: true in their
                          CSIDriver instance. Other volumes are always re-labelled recursively.
                          "MountOption" value is allowed only when SELinuxMount feature gate is enabled.

                          If not specified and SELinuxMount feature gate is enabled, "MountOption" is used.
                          If not specified and SELinuxMount feature gate is disabled, "MountOption" is used for ReadWriteOncePod volumes
                          and "Recursive" for all other volumes.

                          This field affects only Pods that have SELinux label set, either in PodSecurityContext or in SecurityContext of all containers.

                          All Pods that use the same volume should use the same seLinuxChangePolicy, otherwise some pods can get stuck in ContainerCreating state.
                          Note that this field cannot be set when spec.os.name is windows.
                        type: string
                      seLinuxOptions:
                        description: |-
                          The SELinux context to be applied to all containers.
                          If unspecified, the container runtime will allocate a random SELinux context for each
                          container.  May also be set in SecurityContext.  If set in
                          both SecurityContext and PodSecurityContext, the value specified in SecurityContext
                          takes precedence for that container.
                          Note that this field cannot be set when spec.os.name is windows.
                        properties:
                          level:
                            description: Level is SELinux level label that applies
                              to the container.
                            type: string
                          role:
                            description: Role is a SELinux role label that applies
                              to the container.
                            type: string
                          type:
                            description: Type is a SELinux type label that applies
                              to the container.
                            type: string
                          user:
                            description: User is a SELinux user label that applies
                              to the container.
                            type: string
                        type: object
                      seccompProfile:
                        description: |-
                          The seccomp options to use by the containers in this pod.
                          Note that this field cannot be set when spec.os.name is windows.
                        properties:
                          localhostProfile:
                            description: |-
                              localhostProfile indicates a profile defined in a file on the node should be used.
                              The profile must be preconfigured on the node to work.
                              Must be a descending path, relative to the kubelet's configured seccomp profile location.
                              Must be set if type is "Localhost". Must NOT be set for any other type.
                            type: string
                          type:
                            description: |-
                              type indicates which kind of seccomp profile will be applied.
                              Valid options are:

                              Localhost - a profile defined in a file on the node should be used.
                              RuntimeDefault - the container runtime default profile should be used.
                              Unconfined - no profile should be applied.
                            type: string
                        required:
                        - type
                        type: object
                      supplementalGroups:
                        description: |-
                          A list of groups applied to the first process run in each container, in
                          addition to the container's primary GID and fsGroup (if specified).  If
                          the SupplementalGroupsPolicy feature is enabled, the
                          supplementalGroupsPolicy field determines whether these are in addition
                          to or instead of any group memberships defined in the container image.
                          If unspecified, no additional groups are added, though group memberships
                          defined in the container image may still be used, depending on the
                          supplementalGroupsPolicy field.
                          Note that this field cannot be set when spec.os.name is windows.
                        items:
                          format: int64
                          type: integer
                        type: array
                        x-kubernetes-list-type: atomic
                      supplementalGroupsPolicy:
                        description: |-
                          Defines how supplemental groups of the first container processes are calculated.
                          Valid values are "Merge" and "Strict". If not specified, "Merge" is used.
                          (Alpha) Using the field requires the SupplementalGroupsPolicy feature gate to be enabled
                          and the container runtime must implement support for this feature.
                          Note that this field cannot be set when spec.os.name is windows.
                        type: string
                      sysctls:
                        description: |-
                          Sysctls hold a list of namespaced sysctls used for the pod. Pods with unsupported
                          sysctls (by the container runtime) might fail to launch.
                          Note that this field cannot be set when spec.os.name is windows.
                        items:
                          description: Sysctl defines a kernel parameter to be set
                          properties:
                            name:
                              description: Name of a property to set
                              type: string
                            value:
                              description: Value of a property to set
                              type: string
                          required:
                          - name
                          - value
                          type: object
                        type: array
                        x-kubernetes-list-type: atomic
                      windowsOptions:
                        description: |-
                          The Windows specific settings applied to all containers.
                          If unspecified, the options within a container's SecurityContext will be used.
                          If set in both SecurityContext and PodSecurityContext, the value specified in SecurityContext takes precedence.
                          Note that this field cannot be set when spec.os.name is linux.
                        properties:
                          gmsaCredentialSpec:
                            description: |-
                              GMSACredentialSpec is where the GMSA admission webhook
                              (https://github.com/kubernetes-sigs/windows-gmsa) inlines the contents of the
                              GMSA credential spec named by the GMSACredentialSpecName field.
                            type: string
                          gmsaCredentialSpecName:
                            description: GMSACredentialSpecName is the name of the
                              GMSA credential spec to use.
                            type: string
                          hostProcess:
                            description: |-
                              HostProcess determines if a container should be run as a 'Host Process' container.
                              All of a Pod's containers must have the same effective HostProcess value
                              (it is not allowed to have a mix of HostProcess containers and non-HostProcess containers).
                              In addition, if HostProcess is true then HostNetwork must also be set to true.
                            type: boolean
                          runAsUserName:
                            description: |-
                              The UserName in Windows to run the entrypoint of the container process.
                              Defaults to the user specified in image metadata if unspecified.
                              May also be set in PodSecurityContext. If set in both SecurityContext and
                              PodSecurityContext, the value specified in SecurityContext takes precedence.
                            type: string
                        type: object
                    type: object
                  priorityClassName:
                    type: string
                  rollingUpdate:
                    description: 'Optional: Configuration for rolling update of all
                      DaemonSet pods'
                    properties:
                      maxUnavailable:
                        type: string
                    type: object
                  tolerations:
                    description: 'Optional: Set tolerations'
                    items:
                      description: |-
                        The pod this Toleration is attached to tolerates any taint that matches
                        the triple <key,value,effect> using the matching operator <operator>.
                      properties:
                        effect:
                          description: |-
                            Effect indicates the taint effect to match. Empty means match all taint effects.
                            When specified, allowed values are NoSchedule, PreferNoSchedule and NoExecute.
                          type: string
                        key:
                          description: |-
                            Key is the taint key that the toleration applies to. Empty means match all taint keys.
                            If the key is empty, operator must be Exists; this combination means to match all values and all keys.
                          type: string
                        operator:
                          description: |-
                            Operator represents a key's relationship to the value.
                            Valid operators are Exists, Equal, Lt, and Gt. Defaults to Equal.
                            Exists is equivalent to wildcard for value, so that a pod can
                            tolerate all taints of a particular category.
                            Lt and Gt perform numeric comparisons (requires feature gate TaintTolerationComparisonOperators).
                          type: string
                        tolerationSeconds:
                          description: |-
                            TolerationSeconds represents the period of time the toleration (which must be
                            of effect NoExecute, otherwise this field is ignored) tolerates the taint. By default,
                            it is not set, which means tolerate the taint forever (do not evict). Zero and
                            negative values will be treated as 0 (evict immediately) by the system.
                          format: int64
                          type: integer
                        value:
                          description: |-
                            Value is the taint value the toleration matches to.
                            If the operator is Exists, the value should be empty, otherwise just a regular string.
                          type: string
                      type: object
                    type: array
                  updateStrategy:
                    default: RollingUpdate
                    enum:
                    - RollingUpdate
                    - OnDelete
                    type: string
                type: object
              dcgm:
                description: |-
                  DCGM defines the spec for the standalone NVIDIA DCGM hostengine. Disabled by default;
                  when disabled, dcgm-exporter uses its embedded nv-hostengine. NOTE: the reused enabled
                  field carries no server-side default and its IsEnabled() treats nil as enabled, so the
                  controller must default nil enabled to disabled here.
                properties:
                  args:
                    description: 'Optional: List of arguments'
                    items:
                      type: string
                    type: array
                  enabled:
                    description: Enabled indicates if deployment of NVIDIA DCGM Hostengine
                      as a separate pod is enabled.
                    type: boolean
                  env:
                    description: 'Optional: List of environment variables'
                    items:
                      description: EnvVar represents an environment variable present
                        in a Container.
                      properties:
                        name:
                          description: Name of the environment variable.
                          type: string
                        value:
                          description: Value of the environment variable.
                          type: string
                      required:
                      - name
                      type: object
                    type: array
                  hostNetwork:
                    description: HostNetwork indicates whether the DCGM pod uses the
                      host's network namespace.
                    type: boolean
                  hostPort:
                    description: 'Deprecated: HostPort represents host port that needs
                      to be bound for DCGM engine (Default: 5555)'
                    format: int32
                    type: integer
                  image:
                    description: NVIDIA DCGM image name
                    pattern: '[a-zA-Z0-9\-]+'
                    type: string
                  imagePullPolicy:
