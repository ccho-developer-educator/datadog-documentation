---
title: Single Step APM Instrumentation
aliases:
- /tracing/trace_collection/single-step-apm
- /tracing/trace_collection/admission_controller/
- /tracing/trace_collection/library_injection_local/
further_reading:
  - link: /tracing/metrics/runtime_metrics/
    tag: Documentation
    text: Enable Runtime Metrics
---
## Overview

Single Step Instrumentation (SSI) for APM installs the Datadog Agent and [instruments][4] your applications in one step, with no additional configuration steps required.

## Compatibility

To see requirements for compatible languages, operating systems, and architectures, see [Single Step Instrumentation compatibility][6].

## Enabling APM on your applications

If you [install or update a Datadog Agent][1] with the **Enable APM Instrumentation** option selected, the Agent is installed and configured to enable APM. This automatically instruments your application, without any additional installation or configuration steps.

The following examples show how it works for each deployment type.

{{< tabs >}}
{{% tab "Linux host or VM" %}}

For an Ubuntu host:

1. Run the one-line installation command:

   ```shell
   DD_API_KEY=<YOUR_DD_API_KEY> DD_SITE="<YOUR_DD_SITE>" DD_APM_INSTRUMENTATION_ENABLED=host DD_APM_INSTRUMENTATION_LIBRARIES="java:1,python:2,js:5,dotnet:3,php:1" DD_ENV=<AGENT_ENV> bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
   ```

   Replace `<YOUR_DD_API_KEY>` with your [Datadog API key][4], `<YOUR_DD_SITE>` with your [Datadog site][3], and `<AGENT_ENV>` with the environment your Agent is installed on (for example, `staging`).
   <div class="alert alert-info">See <a href=#advanced-options>Advanced options</a> for more options.</div>
2. Start a new shell session.
3. Restart the services on the host or VM.

[3]: /getting_started/site/
[4]: https://app.datadoghq.com/organization-settings/api-keys
[5]: /tracing/software_catalog/
[7]: https://github.com/DataDog/dd-trace-java/releases
[8]: https://github.com/DataDog/dd-trace-js/releases
[9]: https://github.com/DataDog/dd-trace-py/releases
[10]: https://github.com/DataDog/dd-trace-dotnet/releases
[11]: https://github.com/DataDog/dd-trace-rb/releases

{{% /tab %}}

{{% tab "Docker" %}}

For a Docker Linux container:

1. Run the one-line installation command:
   ```shell
   DD_APM_INSTRUMENTATION_ENABLED=docker DD_APM_INSTRUMENTATION_LIBRARIES="java:1,python:2,js:5,dotnet:3,php:1" DD_NO_AGENT_INSTALL=true bash -c "$(curl -L https://install.datadoghq.com/scripts/install_script_agent7.sh)"
   ```
2. Configure the Agent in Docker:
   ```shell
   docker run -d --name dd-agent \
     -e DD_API_KEY=<YOUR_DD_API_KEY> \
     -e DD_APM_ENABLED=true \
     -e DD_ENV=<AGENT_ENV> \
     -e DD_APM_NON_LOCAL_TRAFFIC=true \
     -e DD_DOGSTATSD_NON_LOCAL_TRAFFIC=true \
     -e DD_APM_RECEIVER_SOCKET=/var/run/datadog/apm.socket \
     -e DD_DOGSTATSD_SOCKET=/var/run/datadog/dsd.socket \
     -v /var/run/datadog:/var/run/datadog \
     -v /var/run/docker.sock:/var/run/docker.sock:ro \
     gcr.io/datadoghq/agent:7
   ```
   Replace `<YOUR_DD_API_KEY>` with your [Datadog API key][5] and `<AGENT_ENV>` with the environment your Agent is installed on (for example, `staging`).
   <div class="alert alert-info">See <a href=#advanced-options>Advanced options</a> for more options.</div>
3. Restart the Docker containers.
4. [Explore the performance observability of your services in Datadog][6].

[5]: https://app.datadoghq.com/organization-settings/api-keys
[6]: /tracing/software_catalog/

{{% /tab %}}

{{% tab "Kubernetes" %}}

**Note**: Single Step Instrumentation for Kubernetes is GA for Agent versions 7.64+, and in Preview for Agent versions <=7.63.

You can enable APM by installing the Agent with either:

- Datadog Operator
- Datadog Helm chart

<div class="alert alert-info">Single Step Instrumentation doesn't instrument applications in the namespace where you install the Datadog Agent. It's recommended to install the Agent in a separate namespace in your cluster where you don't run your applications.</div>

### Requirements

- Kubernetes v1.20+
- [`Helm`][1] for deploying the Datadog Operator.
- [`Kubectl` CLI][2] for installing the Datadog Agent.

{{< collapse-content title="Installing with Datadog Operator" level="h4" >}}
Follow these steps to enable Single Step Instrumentation across your entire cluster with the Datadog Operator. This automatically sends traces for all applications in the cluster that are written in supported languages.

**Note**: To configure Single Step Instrumentation for specific namespace or pods, see [Advanced options](#advanced-options).

To enable Single Step Instrumentation with the Datadog Operator:

1. Install the [Datadog Operator][36] with Helm:
   ```shell
   helm repo add datadog https://helm.datadoghq.com
   helm repo update
   helm install my-datadog-operator datadog/datadog-operator
   ```
2. Create a Kubernetes secret to store your Datadog [API key][10]:
   ```shell
   kubectl create secret generic datadog-secret --from-literal api-key=$DD_API_KEY
   ```
3. Create `datadog-agent.yaml` with the spec of your Datadog Agent deployment configuration. The simplest configuration is as follows:
   ```yaml
   apiVersion: datadoghq.com/v2alpha1
   kind: DatadogAgent
   metadata:
     name: datadog
   spec:
     override:
       clusterAgent:
         image:
           tag: 7.64.1
     global:
       site: <DATADOG_SITE>
       tags:
         - env:<AGENT_ENV>
       credentials:
         apiSecret:
           secretName: datadog-secret
           keyName: api-key
     features:
       apm:
         instrumentation:
           enabled: true
           targets:
             - name: "default-target"
               ddTracerVersions:
                 java: "1"
                 dotnet: "3"
                 python: "2"
                 js: "5"
                 php: "1"
   ```
   Replace `<DATADOG_SITE>` with your [Datadog site][12] and `<AGENT_ENV>` with the environment your Agent is installed on (for example, `env:staging`).
   <div class="alert alert-info">See <a href=#advanced-options>Advanced options</a> for more options.</div>

4. Run the following command:
   ```shell
   kubectl apply -f /path/to/your/datadog-agent.yaml
   ```
5. After waiting a few minutes for the Datadog Cluster Agent changes to apply, restart your applications.
{{< /collapse-content >}}

{{< collapse-content title="Installing with Helm" level="h4" >}}
Follow these steps to enable Single Step Instrumentation across your entire cluster with Helm. This automatically sends traces for all applications in the cluster that are written in supported languages.

**Note**: To configure Single Step Instrumentation for specific namespace or pods, see [Advanced options](#advanced-options).

To enable Single Step Instrumentation with Helm:

1. Add the Helm Datadog repo:
   ```shell
    helm repo add datadog https://helm.datadoghq.com
    helm repo update
    ```
2. Create a Kubernetes secret to store your Datadog [API key][10]:
   ```shell
   kubectl create secret generic datadog-secret --from-literal api-key=$DD_API_KEY
   ```
3. Create `datadog-values.yaml` and add the following configuration:
   ```
   datadog:
    apiKeyExistingSecret: datadog-secret
    site: <DATADOG_SITE>
    tags:
         - env:<AGENT_ENV>
    apm:
      instrumentation:
         enabled: true
         targets:
           - name: "default-target"
             ddTracerVersions:
               java: "1"
               dotnet: "3"
               python: "2"
               js: "5"
               php: "1"
   ```
   Replace `<DATADOG_SITE>` with your [Datadog site][12] and `<AGENT_ENV>` with the environment your Agent is installed on (for example, `env:staging`).

   <div class="alert alert-info">See <a href=#advanced-options>Advanced options</a> for more options.</div>

4. Run the following command:
   ```shell
   helm install datadog-agent -f datadog-values.yaml datadog/datadog
   ```
5. After waiting a few minutes for the Datadog Cluster Agent changes to apply, restart your applications.

{{< /collapse-content >}}

[1]: https://v3.helm.sh/docs/intro/install/
[2]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
[9]: https://github.com/DataDog/helm-charts/tree/master/charts/datadog-operator
[10]: https://app.datadoghq.com/organization-settings/api-keys
[11]: https://app.datadoghq.com/organization-settings/application-keys
[12]: /getting_started/site
[13]: https://v3.helm.sh/docs/intro/install/
[36]: https://github.com/DataDog/helm-charts/tree/master/charts/datadog-operator

{{% /tab %}}
{{< /tabs >}}

After you complete these steps, you may want to enable [runtime metrics][2] or view observability data from your application in the [Software Catalog][3].

## Advanced options

When you run the one-line installation command, there are a few options to customize your experience:

{{< tabs >}}
{{% tab "Linux host or VM" %}}

### `DD_APM_INSTRUMENTATION_LIBRARIES` - customizing APM libraries

By default, Java, Python, Ruby, Node.js, PHP and .NET Core Datadog APM libraries are installed when `DD_APM_INSTRUMENTATION_ENABLED` is set. `DD_APM_INSTRUMENTATION_LIBRARIES` is used to override which libraries are installed. The value is a comma-separated string of colon-separated library name and version pairs.

Example values for `DD_APM_INSTRUMENTATION_LIBRARIES`:

- `DD_APM_INSTRUMENTATION_LIBRARIES="java:1"` - install only the Java Datadog APM library pinned to the major version 1 release line.
- `DD_APM_INSTRUMENTATION_LIBRARIES="java:1,python:2"` - install only the Java and Python Datadog APM libraries pinned to the major versions 1 and 2 respectively.
- `DD_APM_INSTRUMENTATION_LIBRARIES="java:1.38.0,python:2.10.5"` - install only the Java and Python Datadog APM libraries pinned to the specific versions 1.38.0 and 2.10.5 respectively.


Available versions are listed in source repositories for each language:

- [Java][8] (`java`)
- [Node.js][9] (`js`)
- [Python][10] (`python`)
- [.NET][11] (`dotnet`)
- [Ruby][12] (`ruby`)
- [PHP][13] (`php`)


[2]: /agent/remote_config
[6]: https://github.com/DataDog/dd-trace-js?tab=readme-ov-file#version-release-lines-and-maintenance
[8]: https://github.com/DataDog/dd-trace-java/releases
[9]: https://github.com/DataDog/dd-trace-js/releases
[10]: https://github.com/DataDog/dd-trace-py/releases
[11]: https://github.com/DataDog/dd-trace-dotnet/releases
[12]: https://github.com/DataDog/dd-trace-rb/releases
[13]: https://github.com/DataDog/dd-trace-php/releases

{{% /tab %}}

{{% tab "Docker" %}}


### `DD_APM_INSTRUMENTATION_LIBRARIES` - customizing APM libraries

By default, Java, Python, Ruby, Node.js and .NET Core Datadog APM libraries are installed when `DD_APM_INSTRUMENTATION_ENABLED` is set. `DD_APM_INSTRUMENTATION_LIBRARIES` is used to override which libraries are installed. The value is a comma-separated string of colon-separated library name and version pairs.

Example values for `DD_APM_INSTRUMENTATION_LIBRARIES`:

- `DD_APM_INSTRUMENTATION_LIBRARIES="java:1"` - install only the Java Datadog APM library pinned to the major version 1 release line.
- `DD_APM_INSTRUMENTATION_LIBRARIES="java:1,python:2"` - install only the Java and Python Datadog APM libraries pinned to the major versions 1 and 2 respectively.
- `DD_APM_INSTRUMENTATION_LIBRARIES="java:1.38.0,python:2.10.5"` - install only the Java and Python Datadog APM libraries pinned to the specific versions 1.38.0 and 2.10.5 respectively.


Available versions are listed in source repositories for each language:

- [Java][8] (`java`)
- [Node.js][9] (`js`)
- [Python][10] (`python`)
- [.NET][11] (`dotnet`)
- [Ruby][12] (`ruby`)
- [PHP][13] (`php`)


[5]: https://app.datadoghq.com/organization-settings/api-keys
[7]: https://github.com/DataDog/dd-trace-js?tab=readme-ov-file#version-release-lines-and-maintenance
[8]: https://github.com/DataDog/dd-trace-java/releases
[9]: https://github.com/DataDog/dd-trace-js/releases
[10]: https://github.com/DataDog/dd-trace-py/releases
[11]: https://github.com/DataDog/dd-trace-dotnet/releases
[12]: https://github.com/DataDog/dd-trace-rb/releases
[13]: https://github.com/DataDog/dd-trace-php/releases

{{% /tab %}}

{{% tab "Kubernetes (Agent v7.64+)" %}}

### Configuring instrumentation for namespaces and pods

By default, Single Step Instrumentation instruments all services in all namespaces in your cluster. Alternatively, you can create targeting blocks with the `targets` label to specify which workloads to instrument and what configurations to apply.

Each target block has the following keys:

| Key                 | Description                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `name`              | The name of the target block. This has no effect on monitoring state and is used only as metadata.                                                                                                                                                                                                                                                                                                           |
| `namespaceSelector` | The namespace(s) to instrument. Specify using one or more of:<br> - `matchNames`: A list of one or more namespace name(s). <br> - `matchLabels`: A list of one or more label(s) defined in `{key,value}` pairs. <br> - `matchExpressions`: A list of namespace selector requirements. <br><br> Namespaces must meet all criteria to match. For more details, see the [Kubernetes selector documentation][3]. |
| `podSelector`       | The pod(s) to instrument. Specify using one or more of: <br> - `matchLabels`: A list of one or more label(s) defined in `{key,value}` pairs. <br> - `matchExpressions`: A list of pod selector requirements. <br><br> Pods must meet all criteria to match. For more details, see the [Kubernetes selector documentation][3].                                                                                |
| `ddTraceVersions`   | The [Datadog APM SDK][2] version to use for each language.                                                                                                                                                                                                                                                                                                                                                   |
| `ddTraceConfigs`    | APM SDK configs that allow setting Unified Service Tags, enabling Datadog products beyond tracing, and customizing other APM settings. [See full list of options][1].                                                                                                                                                                                                                                        |



The file you need to configure depends on how you enabled Single Step Instrumentation:
- If you enabled SSI with Datadog Operator, edit `datadog-agent.yaml`.
- If you enabled SSI with Helm, edit `datadog-values.yaml`.

#### Example configurations

Review the following examples demonstrating how to select specific services:

{{< collapse-content title="Example 1: Enable all namespaces except one" level="h4" >}}

This configuration:
- enables APM for all namespaces except the `jenkins` namespace.
- instructs Datadog to instrument the Java applications with the default Java APM SDK and Python applications with `v.3.1.0` of the Python APM SDK.

{{< highlight yaml "hl_lines=4-10" >}}
   apm:
     instrumentation:
       enabled: true
       disabledNamespaces:
         - "jenkins"
       targets:
         - name: "all-remaining-services"
           ddTraceVersions:
             java: "default"
             python: "3.1.0"
{{< /highlight >}}

{{< /collapse-content >}}

{{< collapse-content title="Example 2: Instrument a subset of namespaces, matching on names and labels" level="h4" >}}

This configuration creates two targets blocks:

- The first block (named `login-service_namespace`):
  - enables APM for services in the namespace `login-service`.
  - instructs Datadog to instrument services in this namespace with the default version of the Java APM SDK.
  - sets environment variables -- `DD_SERVICE`, `DD_ENV`, and `DD_PROFILING_ENABLED` -- for this target group.
- The second block (named `billing-service_apps`)
  - enables APM for services in the namespace(s) with label `app:billing-service`.
  - instructs Datadog to instrument this set of services with `v3.1.0` of the Python APM SDK.
  - sets environment variables -- `DD_SERVICE` and `DD_ENV` -- for this target group.

{{< highlight yaml "hl_lines=4-28" >}}
  apm:
    instrumentation:
      enabled: true
      targets:
        - name: "login-service_namespace"
          namespaceSelector:
            matchNames:
              - "login-service"
          ddTraceVersions:
            java: "default"
          ddTraceConfigs:
            - name: "DD_SERVICE"
              value: "login-service"
            - name: "DD_ENV"
              value: "prod"
            - name: "DD_PROFILING_ENABLED"  ## profiling is enabled for all services in this namespace
              value: "auto"
        - name: "billing-service_apps"
          namespaceSelector:
            matchLabels:
              app: "billing-service"
          ddTraceVersions:
            python: "3.1.0"
          ddTraceConfigs:
            - name: "DD_SERVICE"
              value: "billing-service"
            - name: "DD_ENV"
              value: "prod
{{< /highlight >}}

{{< /collapse-content >}}

{{< collapse-content title="Example 3: Instrument different workloads with different tracers" level="h4" >}}

This configuration does the following:
- enables APM for pods with the following labels:
  - `app:db-user`, which marks pods running the `db-user` application.
  - `webserver:routing`, which marks pods running the `request-router` application.
- instructs Datadog to use the default versions of the Datadog Tracer SDKs.
- sets several Datadog environment variables to apply to each target group.

{{< highlight yaml "hl_lines=4-28" >}}
   apm:
     instrumentation:
       enabled: true
       targets:
         - name: "db-user"
           podSelector:
             matchLabels:
               app: "db-user"
           ddTraceVersions:
             java: "default"
           ddTraceConfigs:   ## trace configs set for services in matching pods
             - name: "DD_SERVICE"
               value: "db-user"
             - name: "DD_ENV"
               value: "prod"
             - name: "DD_DSM_ENABLED"
               value: "true"
         - name: "user-request-router"
           podSelector:
             matchLabels:
               webserver: "user"
           ddTraceVersions:
             php: "default"
           ddTraceConfigs:
             - name: "DD_SERVICE"
               value: "user-request-router"
             - name: "DD_ENV"
               value: "prod
{{< /highlight >}}

{{< /collapse-content >}}

{{< collapse-content title="Example 4: Instrument a pod within a namespace" level="h4" >}}

This configuration:
- enables APM for pods labeled `app:password-resolver` inside the `login-service` namespace.
- instructs Datadog to use the default version of the Datadog Java Tracer SDK.
- sets several Datadog environment variables to apply to this target.

{{< highlight yaml "hl_lines=4-28" >}}
   apm:
     instrumentation:
       enabled: true
       targets:
         - name: "login-service-namespace"
           namespaceSelector:
             matchNames:
               - "login-service"
           podSelector:
             matchLabels:
               app: "password-resolver"
           ddTraceVersions:
             java: "default"
           ddTraceConfigs:
             - name: "DD_SERVICE"
               value: "password-resolver"
             - name: "DD_ENV"
               value: "prod"
             - name: "DD_PROFILING_ENABLED"
               value: "auto"
{{< /highlight >}}

{{< /collapse-content >}}

{{< collapse-content title="Example 5: Instrument a subset of pods using <code>matchExpressions</code>" level="h4" >}}

This configuration enables APM for all pods except those that have either of the labels `app=app1` or `app=app2`.

{{< highlight yaml "hl_lines=4-28" >}}
   apm:
     instrumentation:
       enabled: true
       targets:
         - name: "default-target"
           matchExpressions:
             - key: app
               operator: NotIn
               values:
               - app1
               - app2
{{< /highlight >}}

{{< /collapse-content >}}

[1]: /getting_started/tagging/unified_service_tagging/?tab=kubernetes
[2]: /tracing/trace_collection/automatic_instrumentation/single-step-apm/compatibility/#tracer-libraries
[3]: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#resources-that-support-set-based-requirements

{{% /tab %}}

{{% tab "Kubernetes (Agent <=v7.63) (Preview)" %}}

**Note**: Single Step Instrumentation for Kubernetes is GA for Agent versions 7.64+, and in Preview for Agent versions <=7.63.

### Enabling or disabling instrumentation for namespaces

You can choose to enable or disable instrumentation for applications in specific namespaces. You can only set enabledNamespaces or disabledNamespaces, not both.

The file you need to configure depends on if you enabled Single Step Instrumentation with Datadog Operator or Helm:

{{< collapse-content title="Datadog Operator" level="h4" >}}

To enable instrumentation for specific namespaces, add `enabledNamespaces` configuration to `datadog-agent.yaml`:

{{< highlight yaml "hl_lines=5-7" >}}
   features:
     apm:
       instrumentation:
         enabled: true
         enabledNamespaces: # Add namespaces to instrument
           - default
           - applications
{{< /highlight >}}

To disable instrumentation for specific namespaces, add `disabledNamespaces` configuration to `datadog-agent.yaml`:

{{< highlight yaml "hl_lines=5-7" >}}
   features:
     apm:
       instrumentation:
         enabled: true
         disabledNamespaces: # Add namespaces to not instrument
           - default
           - applications
{{< /highlight >}}

{{< /collapse-content >}}

{{< collapse-content title="Helm" level="h4" >}}

To enable instrumentation for specific namespaces, add `enabledNamespaces` configuration to `datadog-values.yaml`:

{{< highlight yaml "hl_lines=5-7" >}}
   datadog:
      apm:
        instrumentation:
          enabled: true
          enabledNamespaces: # Add namespaces to instrument
             - namespace_1
             - namespace_2
{{< /highlight >}}

To disable instrumentation for specific namespaces, add `disabledNamespaces` configuration to `datadog-values.yaml`:

{{< highlight yaml "hl_lines=5-7" >}}
   datadog:
      apm:
        instrumentation:
          enabled: true
          disabledNamespaces: # Add namespaces to not instrument
            - namespace_1
            - namespace_2
{{< /highlight >}}

{{< /collapse-content >}}


### Specifying tracing library versions

<div class="alert alert-info">Starting with Datadog Cluster Agent v7.52.0+, you can automatically instrument a subset of your applications, based on the tracing libraries you specify.</div>

Specify Datadog tracing libraries and their versions to automatically instrument applications written in those languages. You can configure this in two ways, which are applied in the following order of precedence:

1. [Specify at the service level](#specifying-at-the-service-level), or
2. [Specify at the cluster level](#specifying-at-the-cluster-level).

**Default**: If you don't specify any library versions and `apm.instrumentation.enabled=true`, applications written in supported languages are automatically instrumented using the latest tracing library versions.

#### Specifying at the service level

To automatically instrument applications in specific pods, add the appropriate language annotation and library version for your application in your pod spec:

| Language | Pod annotation                                                        |
| -------- | --------------------------------------------------------------------- |
| Java     | `admission.datadoghq.com/java-lib.version: "<CONTAINER IMAGE TAG>"`   |
| Node.js  | `admission.datadoghq.com/js-lib.version: "<CONTAINER IMAGE TAG>"`     |
| Python   | `admission.datadoghq.com/python-lib.version: "<CONTAINER IMAGE TAG>"` |
| .NET     | `admission.datadoghq.com/dotnet-lib.version: "<CONTAINER IMAGE TAG>"` |
| Ruby     | `admission.datadoghq.com/ruby-lib.version: "<CONTAINER IMAGE TAG>"`   |
| PHP      | `admission.datadoghq.com/php-lib.version: "<CONTAINER IMAGE TAG>"`    |

Replace `<CONTAINER IMAGE TAG>` with the desired library version. Available versions are listed in the [Datadog container registries](#container-registries) and tracer source repositories for each language:

- [Java][34]
- [Node.js][35]
- [Python][36]
- [.NET][37]
- [Ruby][38]
- [PHP][39]

<div class="alert alert-warning">Exercise caution when using the <code>latest</code> tag, as major library releases may introduce breaking changes.</div>

For example, to automatically instrument Java applications:

{{< highlight yaml "hl_lines=10" >}}
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    # ...
spec:
  template:
    metadata:
      annotations:
        admission.datadoghq.com/java-lib.version: "<CONTAINER IMAGE TAG>"
    spec:
      containers:
        - # ...
{{< /highlight >}}

#### Specifying at the cluster level

If you don't enable automatic instrumentation for specific pods using annotations, you can specify which languages to instrument across the entire cluster using the Single Step Instrumentation configuration. When `apm.instrumentation.libVersions` is set, only applications written in the specified languages will be instrumented, using the specified library versions.

The file you need to configure depends on if you enabled Single Step Instrumentation with Datadog Operator or Helm:

{{< collapse-content title="Datadog Operator" level="h4" >}}

For example, to instrument .NET, Python, and Node.js applications, add the following configuration to your `datadog-agent.yaml` file:

{{< highlight yaml "hl_lines=5-8" >}}
   features:
     apm:
       instrumentation:
         enabled: true
         libVersions: # Add any libraries and versions you want to set
            dotnet: "3.2.0"
            python: "1.20.6"
            js: "4.17.0"
{{< /highlight >}}

{{< /collapse-content >}}

{{< collapse-content title="Helm" level="h4" >}}

For example, to instrument .NET, Python, and Node.js applications, add the following configuration to your `datadog-values.yaml` file:

{{< highlight yaml "hl_lines=5-8" >}}
   datadog:
     apm:
       instrumentation:
         enabled: true
         libVersions: # Add any libraries and versions you want to set
            dotnet: "3.2.0"
            python: "1.20.6"
            js: "4.17.0"
{{< /highlight >}}

{{< /collapse-content >}}


#### Container registries

Datadog publishes instrumentation libraries images on gcr.io, Docker Hub, and Amazon ECR:

| Language | gcr.io                                    | hub.docker.com                                    | gallery.ecr.aws                                  |
| -------- | ----------------------------------------- | ------------------------------------------------- | ------------------------------------------------ |
| Java     | [gcr.io/datadoghq/dd-lib-java-init][15]   | [hub.docker.com/r/datadog/dd-lib-java-init][16]   | [gallery.ecr.aws/datadog/dd-lib-java-init][17]   |
| Node.js  | [gcr.io/datadoghq/dd-lib-js-init][18]     | [hub.docker.com/r/datadog/dd-lib-js-init][19]     | [gallery.ecr.aws/datadog/dd-lib-js-init][20]     |
| Python   | [gcr.io/datadoghq/dd-lib-python-init][21] | [hub.docker.com/r/datadog/dd-lib-python-init][22] | [gallery.ecr.aws/datadog/dd-lib-python-init][23] |
| .NET     | [gcr.io/datadoghq/dd-lib-dotnet-init][24] | [hub.docker.com/r/datadog/dd-lib-dotnet-init][25] | [gallery.ecr.aws/datadog/dd-lib-dotnet-init][26] |
| Ruby     | [gcr.io/datadoghq/dd-lib-ruby-init][27]   | [hub.docker.com/r/datadog/dd-lib-ruby-init][28]   | [gallery.ecr.aws/datadog/dd-lib-ruby-init][29]   |
| PHP      | [gcr.io/datadoghq/dd-lib-php-init][30]    | [hub.docker.com/r/datadog/dd-lib-php-init][31]    | [gallery.ecr.aws/datadog/dd-lib-php-init][32]    |

The `DD_ADMISSION_CONTROLLER_AUTO_INSTRUMENTATION_CONTAINER_REGISTRY` environment variable in the Datadog Cluster Agent configuration specifies the registry used by the Admission Controller. The default value is `gcr.io/datadoghq`.

You can pull the tracing library from a different registry by changing it to `docker.io/datadog`, `public.ecr.aws/datadog`, or another URL if you are hosting the images in a local container registry.

For instructions on changing your container registry, see [Changing Your Container Registry][33].

[1]: https://helm.sh/
[2]: https://kubernetes.io/docs/tasks/tools/
[3]: https://app.datadoghq.com/organization-settings/api-keys
[4]: /getting_started/site/
[5]: https://github.com/DataDog/helm-charts/tree/master/charts/datadog-operator
[6]: /getting_started/site
[15]: http://gcr.io/datadoghq/dd-lib-java-init
[16]: http://hub.docker.com/r/datadog/dd-lib-java-init
[17]: http://gallery.ecr.aws/datadog/dd-lib-java-init
[18]: http://gcr.io/datadoghq/dd-lib-js-init
[19]: http://hub.docker.com/r/datadog/dd-lib-js-init
[20]: http://gallery.ecr.aws/datadog/dd-lib-js-init
[21]: http://gcr.io/datadoghq/dd-lib-python-init
[22]: http://hub.docker.com/r/datadog/dd-lib-python-init
[23]: http://gallery.ecr.aws/datadog/dd-lib-python-init
[24]: http://gcr.io/datadoghq/dd-lib-dotnet-init
[25]: http://hub.docker.com/r/datadog/dd-lib-dotnet-init
[26]: http://gallery.ecr.aws/datadog/dd-lib-dotnet-init
[27]: http://gcr.io/datadoghq/dd-lib-ruby-init
[28]: http://hub.docker.com/r/datadog/dd-lib-ruby-init
[29]: http://gallery.ecr.aws/datadog/dd-lib-ruby-init
[30]: http://gcr.io/datadoghq/dd-lib-php-init
[31]: http://hub.docker.com/r/datadog/dd-lib-php-init
[32]: http://gallery.ecr.aws/datadog/dd-lib-php-init
[33]: /containers/guide/changing_container_registry/
[34]: https://github.com/DataDog/dd-trace-java/releases
[35]: https://github.com/DataDog/dd-trace-js/releases
[36]: https://github.com/DataDog/dd-trace-py/releases
[37]: https://github.com/DataDog/dd-trace-dotnet/releases
[38]: https://github.com/DataDog/dd-trace-rb/releases
[39]: https://github.com/DataDog/dd-trace-php/releases

{{% /tab %}}
{{< /tabs >}}

## Removing Single Step APM instrumentation from your Agent

If you don't want to collect trace data for a particular service, host, VM, or container, complete the following steps:

### Removing instrumentation for specific services

To remove APM instrumentation and stop sending traces from a specific service, follow these steps:

{{< tabs >}}
{{% tab "Linux host or VM" %}}

1. Add the `DD_INSTRUMENT_SERVICE_WITH_APM` environment variable to the service startup command:

   ```shell
   DD_INSTRUMENT_SERVICE_WITH_APM=false <service_start_command>
   ```
2. Restart the service.

{{% /tab %}}

{{% tab "Docker" %}}

1. Add the `DD_INSTRUMENT_SERVICE_WITH_APM` environment variable to the service startup command:
   ```shell
   docker run -e DD_INSTRUMENT_SERVICE_WITH_APM=false <service_start_command>
   ```
2. Restart the service.
{{% /tab %}}

{{% tab "Kubernetes" %}}

**Note**: Single Step Instrumentation for Kubernetes is GA for Agent versions 7.64+, and in Preview for Agent versions <=7.63.

#### Using workload selection (recommended)

With workload selection, you can enable and disable tracing for specific applications. [See configuration details here](#advanced-options).

#### Using the Datadog Admission Controller

As an alternative, or for a version of the agent that does not support workload selection, you can also disable pod mutation by adding a label to your pod.

<div class="alert alert-warning">In addition to disabling SSI, the following steps disable other mutating webhooks. Use with caution.</div>

1. Set the `admission.datadoghq.com/enabled:` label to `"false"` for the pod spec:
   ```yaml
   spec:
     template:
       metadata:
         labels:
           admission.datadoghq.com/enabled: "false"
   ```
2. Apply the configuration:
   ```shell
   kubectl apply -f /path/to/your/deployment.yaml
   ```
3. Restart the services you want to remove instrumentation for.

{{% /tab %}}
{{< /tabs >}}

### Removing APM for all services on the infrastructure

To stop producing traces, uninstall APM and restart the infrastructure:

{{< tabs >}}
{{% tab "Linux host or VM" %}}

1. Run:
   ```shell
   dd-host-install --uninstall
   ```
2. Restart the services on the host or VM.

{{% /tab %}}

{{% tab "Docker" %}}

1. Run:
   ```shell
   dd-container-install --uninstall
   ```
2. Restart Docker:
   ```shell
   systemctl restart docker
   ```
   Or use the equivalent for your environment.

{{% /tab %}}

{{% tab "Kubernetes" %}}

**Note**: Single Step Instrumentation for Kubernetes is GA for Agent versions 7.64+, and in Preview for Agent versions <=7.63.

The file you need to configure depends on if you enabled Single Step Instrumentation with Datadog Operator or Helm:

{{< collapse-content title="Datadog Operator" level="h4" >}}

1. Set `instrumentation.enabled=false` in `datadog-agent.yaml`:
   ```yaml
   features:
     apm:
       instrumentation:
         enabled: false
   ```

2. Deploy the Datadog Agent with the updated configuration file:
   ```shell
   kubectl apply -f /path/to/your/datadog-agent.yaml
   ```
{{< /collapse-content >}}

{{< collapse-content title="Helm" level="h4" >}}

1. Set `instrumentation.enabled=false` in `datadog-values.yaml`:
   ```yaml
   datadog:
     apm:
       instrumentation:
         enabled: false
   ```

2. Run the following command:
   ```shell
   helm upgrade datadog-agent -f datadog-values.yaml datadog/datadog
   ```
{{< /collapse-content >}}

{{% /tab %}}
{{< /tabs >}}

## Troubleshooting

### Single Step Instrumentation is not taking effect

Single Step Instrumentation automatically disables when it detects [custom instrumentation][7] in your application. If you want to use SSI, you'll need to:

1. Remove any existing custom instrumentation code.
1. Restart your application.

## Further reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: https://app.datadoghq.com/account/settings/agent/latest
[2]: /tracing/metrics/runtime_metrics/
[3]: /tracing/software_catalog/
[4]: /tracing/glossary/#instrumentation
[5]: /containers/cluster_agent/admission_controller/
[6]: /tracing/trace_collection/automatic_instrumentation/single-step-apm/compatibility
[7]: /tracing/trace_collection/custom_instrumentation/
