---
title: "Kube-Policies: Guardrails for Apps Running in Kubernetes"
datePublished: Tue Jan 28 2025 08:00:00 GMT+0000 (Coordinated Universal Time)
cuid: cm8q3lk8i000b09la5j3t1asi
slug: kube-policies-guardrails-for-apps-running-in-kubernetes
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1743004033407/5d8de43c-5841-491e-8b9d-ceabc9f66d74.png

---


## Introduction

In the fast-paced world of cloud computing, security shouldn't be a barrier, but a facilitator. Our Compute Security team's philosophy embraces creating guardrails instead of gates, guiding innovation securely without compromising speed. This blog series unveils our journey of establishing security guardrails in Kubernetes environments, showcasing the challenges we faced and the solutions we devised.

## Problem statement

Kubernetes, a powerhouse for orchestrating modern applications, was crafted with extensibility as a core tenet. However, its default configurations often favor rapid deployment over security. This oversight leaves applications vulnerable, as over-reliance on defaults invites malicious actors. With developers focused on innovation, security configurations often become an afterthought.

We recognized the need to address this gap. Kubernetes provides built-in features like [admission controllers](http://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/), which evolved from Pod Security Policies (PSPs) to the more robust [Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/) Controller. Yet, our diverse client platforms demanded an even more adaptive and powerful solution. Enter our decision to pivot to an abstraction layer built atop the widely celebrated [Open Policy Agent (OPA)](https://www.openpolicyagent.org/).

## Requirements

Each client platform we serve presents unique challenges, demanding flexible policies. To retire PSPs, we established key requirements for our security framework:

1. **Policy promotion process:** We needed a mechanism for "dry-runs" to evaluate policy impacts without disrupting client environments.

2. **Minimal user disruption:** The most effective security tools are those that do their job transparently. With that in mind, we wanted to be able to mutate resources as needed so user intervention to fix resource misconfigurations was minimal.

3. **Robust testing framework:** Admission controllers are, by design, gatekeepers of everything that gets deployed to the clusters, so any issues with them could become a catastrophic cluster failure. That’s why a solid testing framework for policy changes was mandatory.

4. **Exception management:** Policies must be able to accommodate exceptions to support innovation without compromising security.

5. **Extensibility:** The policy engine must be able to be extended as needed. One example could be an integration with a third party system to manage notifications about certain policy decisions.

6. **Observability:** Being able to understand the state of the admission controller at all times was of the utmost importance. Some examples of questions we needed to answer:

   a. How many requests are being denied and why.

   b. What does resource consumption look like?

   c. Is the admission controller up or down?

7. **Security:** The policy engine must embed all the best practices suggested by the Compute Security team. Examples like offering strong provenance assurance, minimizing the chances of a privilege escalation attack, compliance with certain frameworks of interest like PCI, SOX, etc are prime examples of these.

## Architecture

We looked into many admission control solutions like [K-Rail](https://github.com/cruise-automation/k-rail), [Gatekeeper](https://github.com/open-policy-agent/gatekeeper) and [Kyverno](https://github.com/kyverno/kyverno), but at the time none of them met all our requirements, nor were they capable of moving as fast as we needed to serve our use cases. Therefore, we decided to build our own solution based on OPA, which we named **_kube-policies_**. A high level architecture diagram is shown next:

![image1](//images.ctfassets.net/1wryd5vd9xez/T9SvPoTJCwycIwsKdnS4X/beb922d52e09803595acaaeddef5cbb7/image1.png)

Some points worth explaining a bit more in the diagram:

1. **Policy types:** We considered resource mutations for our use cases in addition to validations to minimize user intervention both during the migration process off PSPs and on regular day to day operations. Anything we can take off our user’s plate while keeping them safe is well worth the investment on our side.

2. **Audit logs:** All major managed cloud providers provide a way to stream Kubernetes audit logs to their built-in logging solutions, which makes this mechanism a great vehicle to surface policy decisions to platform users.

3. **External dependencies:** Both OPA and Kubernetes were built with extensibility in mind, which makes integrations with third party systems relatively straightforward. A few examples of these integrations could be using Datadog to observe metrics, Slack for notifications, KMS to help with signature validations, etc.

## Policy promotion

Our strategy involves gradual policy enforcement to minimize runtime disruptions. Policies undergo a meticulous journey through monitoring, warning, and denial stages. This staged approach allows observation in lower environments before progressing to production, ensuring minimal disruption and maximum learning:

### Validating policies

1. **Monitoring**: In this stage, a policy is applied, but does not enforce the control. It just updates the kubernetes audit logs with annotations that reflect policy decisions. This helps test our assumptions about how the policy works in the platform and allows platform teams to understand how a new policy may affect their workloads without any impact.

   ```
   package validating

   monitor[decision] {
       kube_policies.privilege_escalation.deny_privilege_escalation[message] with data.allowlist as allowlist

       decision := message
   }
   ```

2. **Warning**: This stage starts showing a warning to the users by using the [Warnings HTTP header.](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#response) In this stage, application owners start receiving recommendations about the changes to meet the policy requirements but without the controls being enforced. This is our first attempt to nudge users as needed after we have better confidence on the desired outcome.

   ```
   package validating

   warn[decision] {
       kube_policies.privilege_escalation.deny_privilege_escalation[message] with data.allowlist as allowlist

       decision := message
   }
   ```

3. **Deny**: As the name suggests, this stage starts blocking the incoming requests if policy requirements are not met to prevent non-compliant resources from being deployed to our clusters.

   ```
   package validating

   deny[decision] {
       kube_policies.privilege_escalation.deny_privilege_escalation[message] with data.allowlist as allowlist

       decision := message
   }
   ```

### Mutating policies

1. **DryRun**: This mode only adds an annotation to the audit logs with the patches that would’ve been applied if mutations were to be made to the input resources. The idea is similar to the monitoring mode in the validating policies.

   ```
   package mutating

   dryrun[patch_code] {
     kube_policies.readonly_root_fs.set_readonly_root_fs[change] with data.allowlist as allowlist

     patch_code := change
   }
   ```

2. **What about “warnings?”** We didn’t add a “warning” stage here because the whole point of mutating policies is to make resources compliant without user intervention, so the job of nudging users in the right direction is done where needed by the validating policies.
3. **Patch**: This is the fully enforced stage for mutating policies. It adds the audit logs annotations with the policy decisions and mutates the incoming resources as needed to bring resources into compliance without requiring user interaction.

   ```
   package mutating

   patch[patch_code] {
     kube_policies.readonly_root_fs.set_readonly_root_fs[change] with data.allowlist as allowlist

     patch_code := change
   }
   ```

Our platforms operate under three types of environments: development, staging, and production. When we decide to promote a policy from monitoring to warning mode, or warning to deny mode, we initiate this transition first in the lower environments. This cautious approach allows us to observe the policy’s impact on workloads without affecting the cluster state or any services.

Only after collecting sufficient data on how a policy change will influence workloads, we promote a policy to a more restrictive phase in higher environments. This promotion framework in conjunction with thorough tests ensures that we provide ample notice to teams and prevent any unexpected disruptions of workloads.

## Policy testing

Before releasing a policy in any mode to our platforms, we conduct an extensive testing exercise to ensure stability and prevent any potential disruption. Kubernetes admission controllers have a lot of power, and a bug can potentially cripple an entire cluster. To reduce this risk, we employ a comprehensive testing strategy, which includes:

1. **Unit tests**: For any policy change, we make sure we have several unit tests that mock the actual admission review objects to cover as many scenarios as possible:

   a. **Valid scenario**: Confirming that compliant resources are accepted.

   b. **Invalid scenario**: Ensure non-compliant resources are denied.

   c. **Edge cases**: Testing boundaries and unusual scenarios.

   d. **Invalid inputs**: Verifying the policy’s robustness against malformed data.

   These tests are an integral part of our development and build process. Our build pipelines are configured to fail if any unit test fails, preventing unintended regressions.

   ```
   # Inputs: Pod Violating all Policies (regular containers)
   # Outputs: Violation Message for All Violating Policies
   # Assert: mutations == Object Containing All Needed Mutations
   test_dryrun_invalid_1 {
     mutations := dryrun with input as test_fixture.invalid_pod

     expected := {
        {
          "op": "add",
          "path": "/spec/containers/0/securityContext/readOnlyRootFilesystem",
          "value": true,
        },
        {
          "op": "add",
          "path": "/spec/containers/0/securityContext/capabilities/drop",
          "value": ["AUDIT_CONTROL", "AUDIT_READ", "AUDIT_WRITE", "BLOCK_SUSPEND", "CHOWN", "DAC_OVERRIDE", "DAC_READ_SEARCH", "FOWNER", "FSETID", "IPC_LOCK", "IPC_OWNER", "KILL", "LEASE", "LINUX_IMMUTABLE", "MAC_ADMIN", "MAC_OVERRIDE", "MKNOD", "NET_ADMIN", "NET_BROADCAST", "NET_RAW", "SETFCAP", "SETGID", "SETPCAP", "SETUID", "SYSLOG", "SYS_ADMIN", "SYS_BOOT", "SYS_CHROOT", "SYS_MODULE", "SYS_NICE", "SYS_PACCT", "SYS_PTRACE", "SYS_RAWIO", "SYS_RESOURCE", "SYS_TIME", "SYS_TTY_CONFIG", "WAKE_ALARM"],
        },
        {
          "op": "add",
          "path": "/spec/containers/0/securityContext/runAsNonRoot",
          "value": true,
        },
     }

     mutations == expected
   }
   ```

2. **End to end tests**: This is another crucial part of our development workflow. We utilize the Kubernetes [e2e testing framework](https://github.com/kubernetes-sigs/e2e-framework) and the [kind](https://kind.sigs.k8s.io/) project to spin up an ephemeral cluster for each new version we build. Within these temporary environments, we deploy actual Kubernetes workloads to observe how **_kube-policies_** behaves in an environment very close to a real one. This allows us to catch any issues that unit tests may miss, providing an additional layer of validation before deployment.

   ```
   func TestCapabilities(t *testing.T) {
     capabilitiesFeat1 := features.New("Valid Pod").
       WithLabel("status", "enabled").
       Assess("No policy violations", func(ctx context.Context, t *testing.T, cfg *envconf.Config) context.Context {
         // Transform will update the pod spec to add drop: ALL, which is required for a valid spec
         transform := func(p *v1.Pod) {
           p.Spec.Containers[0].SecurityContext.Capabilities = kube.DropCapabilitiesContainerSecurityContext(v1.Capability("ALL"))
         }

         // Create test pod with valid defaults (no policy violations)
         pod, err := kube.CreateTestPod(ctx, *cfg, "caps-valid-pod", randomLength, transform)
         if err != nil {
           t.Fatalf("couldn't create test pod: %v\n", err)
         }

         // Filters k8s audit logs using the success criteria for this particular use case
         filter := func(annotations kube.AuditAnnotations) error {
           if strings.Contains(annotations.Admission.Denied, capabilitiesValidation) || strings.Contains(annotations.Admission.Patches, capabilitiesMutation) || net.RequestDenied(annotations.Status.Code) {
             return fmt.Errorf("client resource %s didn't match the proper policy %s", pod.Name, capabilitiesValidation)
           }
           return nil
         }

         // Checks k8s audit logs and analyzes policy decisions to determine whether this test case was handled successfully
         err = kube.CheckPolicyDecisions(ctx, clusterName, pod.Name, filter)
         if err != nil {
           t.Logf("error analyzing policy decisions: %v\n", err)
           t.Fail()
         }

         return ctx
       }).Feature()
   }
   ```

3. **Manual testing:** For the sake of simplicity and experimentation we also leverage OPA running locally in server mode, which allows us to send an AdmissionReview object to this endpoint and inspect responses manually.

   ```
   # Run OPA in server mode in localhost

   docker run --rm -p 8181:8181 $IMAGE run --server -b $BUNDLE --log-level debug

   # Send AdmissionReview request (data.json file)

   curl localhost:8181/v0/data/system/validating -d @invalid_cron_unset/data.json -H 'Content-Type: application/json' | jq .
   ```

4. **Code quality:** Beyond these tests, we have several other tests in our build process to ensure quality, consistency, and maintainability to our updates:

   a. Static Analysis using [CodeQL](https://codeql.github.com/)

   b. Linting tools:

      &nbsp;&nbsp;&nbsp;&nbsp;i. [golangci-lint](https://github.com/golangci/golangci-lint) for Go code.

      &nbsp;&nbsp;&nbsp;&nbsp;ii. [helm lint](https://helm.sh/docs/helm/helm_lint/) for Helm charts.

      &nbsp;&nbsp;&nbsp;&nbsp;iii. [regal](https://github.com/StyraInc/regal) for Rego policies.

   c. Formatting tools:

      &nbsp;&nbsp;&nbsp;&nbsp;i. [gofmt](https://pkg.go.dev/cmd/gofmt) for Go code.

      &nbsp;&nbsp;&nbsp;&nbsp;ii. [opa fmt](https://www.openpolicyagent.org/docs/latest/cli/#opa-fmt) for rego policies.

   d. Rego policy validation with [opa check –strict](https://www.openpolicyagent.org/docs/latest/cli/#opa-check)

## Policy failures

1. **Preventing catastrophic failures**: While kube-policies is instrumental in enforcing security guardrails within our Kubernetes clusters, they can inadvertently become a single point of failure if not managed properly. There are a number of scenarios when admission controllers can cause catastrophic cluster failures, and it is important to recognize these scenarios to reduce these risks.

   a. The Container Network Interface (CNI) plugin is a prime example of a potential catastrophic failure. If kube-policies prevents updates to a CNI plugin, the connectivity in the cluster can be compromised, rendering the cluster unusable.

   b. System namespaces (kube-system, kube-node-lease) also host critical components. If an admission controller fails due to unavailability or high latency, it can block essential cluster operations. This can destabilize the entire cluster.

2. **Circular dependencies**: This is another failure scenario with potentially catastrophic consequences given that kube-policies is in the hot path of everything that gets deployed to the cluster. What would happen if a policy failure was causing an application outage and we needed to rollback certain changes but the current active policies don’t allow this operation?
3. **Admission control failures:** What would happen if the policy engine stopped working? Should we allow the requested resource to be deployed or not? This decision will obviously depend on your risk tolerance, but in general can be handled using the failure modes in the Kubernetes webhook configurations, which will dictate whether you fail open or closed.

   ```
   - admissionReviewVersions:
     - v1
       clientConfig:
       caBundle: $REDACTED
       service:
       name: kube-policies
       namespace: kube-policies
       path: /v0/data/system/validating
       port: 443
       failurePolicy: Fail # Can also be Ignore
       matchPolicy: Equivalent
       name: kube-policies-system-validating.example.com
       namespaceSelector:
       matchExpressions:
       - key: kubernetes.io/metadata.name
         operator: In
         values:
         - kube-system
   ```

## Policy exceptions

In a shared platform, exception management becomes a necessary functionality to balance experimentation, speed, and security. While kube-policies enforces security guardrails, there are scenarios where exceptions are essential to maintain agility and business continuity.

1.  **Flexible Schema:** The schema of policy exceptions was designed with extensibility in mind, supporting self-service or automated exception management. Exceptions are organized hierarchically - by environment, policy, namespace, and container name. This structure not only provides all the metadata we need to understand the exception in place, but also lets us get exceptions in constant time, which contributes to the performance of the policy evaluation process.

    ```
    allowlist := {
      "dev": {
        "host_network": {
          "sysdig": {"sysdig-agent"},
        }
      }
    }
    # Schema: {ENV: {POLICY: {NAMESPACE: {CONTAINER_NAME}}}}
    ```

2.  **Additional Exception Mechanisms:** Beyond regular workload exceptions, kube-policies provides two other methods to grant exceptions, ensuring critical operations and components can function without any interruptions.

    a. **Breakglass Allowlist:** From time to time, cluster operators need to perform operations for cluster stability or to troubleshoot resources. The breakglass allowlist enables these operators to bypass policies when using their cluster context with elevated privileges. This allowlist is configured to bypass policies when using elevated privileges, and the list is based on groups, username, and username prefixes.

    b. **Critical Allowlist:** Certain cluster components are so vital that any interruption could render the cluster unusable. Examples include `aws-node`, and `kube-proxy`. These resources are added to the critical allowlist, ensuring that they bypass all policies. This approach of defining a critical allowlist prevents any risk of cluster failure due to a policy issue or a circular dependency.

## Observability

We hold kube-policies to a high standard of reliability and performance to ensure its only impact on cluster workloads is to validate their security configurations. Our team monitors health and performance metrics of our workloads along with the results of our webhooks.

Our monitors alert us if our Pods or Containers are in unexpected states or if there is a high ratio of CPU usage/limit, memory usage/limit, or if our HTTP request duration is too high. To help us scale with our traffic, we use a [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) and since this addition haven’t had any issues with response time to our users.

Even more interesting are the metrics we monitor from the results of our admission controllers. We track our total requests along with our rejection count and monitor the rejection percentage. A high rejection percentage could mean a Pod or Container has been popped and a malicious user is attempting to elevate permissions. Equally worryingly, it could also mean our admission controller is having an error or that there is a bug in one of our policies. No matter the circumstance, a high level of denials from our admission controller is cause for concern, even if the only issue is just a new application that needs help fixing its configuration or getting an exception.

### Audit log annotations

[Kubernetes audit logs](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/) provide a wealth of information about all Kubernetes API server requests. Kubernetes audit log events are configured by an [audit log policy.](https://kubernetes.io/docs/tasks/debug/debug-cluster/audit/#audit-policy) Audit log events enable cluster administrators to answer the following questions about a given request:

1.  What happened?

2.  When did it happen?

3.  Who initiated it?

4.  On what did it happen?

5.  Where was it observed?

6.  From where was it initiated?

7.  To where was it going?

Given the diverse configurations across the business units we support, our strategy to serve them in the best possible way and reduce friction in the configuration process was to meet them where they are. This means utilizing their existing log aggregation strategies to handle this new stream of information. Then it is the responsibility of their respective cluster operators or security personnel to make the best use of this data on a daily basis. It doesn’t mean we just hand off the solution, though – we are always there as a consulting partner to help our clients get the maximum benefit from these solutions so they can improve their overall security posture.

We integrated OPA decisions in the native Kubernetes audit log mechanism via [custom audit annotations](https://github.com/kubernetes/kubernetes/blob/bdd2e476959070b1698ba91c2f6e0cf3688338fd/pkg/apis/admission/types.go#L140). This setup helps cluster operators or security personnel query this stream of data to find information about the admission of resources in the Kubernetes clusters where the kube-policies application was deployed. Since we are sending structured logs to the platform-specific log aggregation system, this audit annotation will be an additional key/value pair that will be part of the Kubernetes audit logs stream. That means that cluster operators can leverage their platform tools to query the following field:

```
"validating-webhook.openpolicyagent.org/key": “value”
```

The key string in the annotation will be an identifier for the rules that were triggered by the admission request, and the possible values are:

- **all_rules:** Comma separated list of values with all the rules that were triggered for a particular admission request.

- **denied:** Comma separated list of values of all the rules that were triggered for a particular admission request, whose decision is to deny the request from being applied in the Kubernetes cluster.

- **monitored:** Comma separated list of values of all the rules that were triggered for a particular admission request but are not active yet. This is useful, for example, to test new rules and audit their decisions before they get promoted to enforce mode.

- **warned:** Comma separated list of values of all warned rules that were triggered for a particular admission request, whose decision was to allow the request but something was found in the request object that requires attention. A useful use case could be Kubernetes API deprecation warnings that aren’t fully gone yet, but need to be handled in a timely manner before an incident occurs.

- **breakglass_authorized:** Breakglass policy authorized, rule checks bypassed. Returns a list of identity types that were authorized.

The value string, on the other hand, contains all the rules messages associated with a specific key separated by a comma so they can be easily retrieved and split up in a programmatic way.

Here is a sample message with these annotations in action:

```
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "allowed": false,
    "auditAnnotations": {
      "all_rules": "deny_host_ipc: Pod Pod/read-only-fs/deny cannot be created with hostIPC enabled., deny_host_network: Pod Pod/read-only-fs/deny cannot be created with hostNetwork enabled., ensure_no_run_as_root: Resource Pod/read-only-fs/deny (container=deny) contains a securityContext specifying run as root.",
      "denied": "deny_host_network: Pod Pod/read-only-fs/deny cannot be created with hostNetwork enabled., ensure_no_run_as_root: Resource Pod/read-only-fs/deny (container=deny) contains a securityContext specifying run as root.",
      "monitored": "deny_host_ipc: Pod Pod/read-only-fs/deny cannot be created with hostIPC enabled.",
      "warned": "warnings from rule"
    },
    "status": {
      "code": 403,
      "message": "deny_host_ipc: Pod Pod/read-only-fs/deny cannot be created with hostIPC enabled., deny_host_network: Pod Pod/read-only-fs/deny cannot be created with hostNetwork enabled., ensure_no_run_as_root: Resource Pod/read-only-fs/deny (container=deny) contains a securityContext specifying run as root.",
      "reason": "Forbidden"
    },
    "uid": "4570d0a2-8eb0-4a38-8053-b93c2cf0758f",
    "warnings": [
      "warnings from rule"
    ]
  }
}
```

There are also different annotations for special cases like default responses (fail-closed, fail-open, default-allow) and breakglass events.

**Fail closed:**

```
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "allowed": false,
    "auditAnnotations": {
      "failing-closed": "Error, failing closed"
    },
    "status": {
      "code": 403,
      "message": "Failing closed",
      "reason": "Forbidden"
    }
  }
}
```

**Fail open:**

```
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "allowed": true,
    "auditAnnotations": {
      "failing-open": "Error, failing open"
    },
    "uid": "4570d0a2-8eb0-4a38-8053-b93c2cf0758f"
  }
}
```

**Default Allowed:**

```
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "allowed": true,
    "auditAnnotations": {
      "default-allow": "Request does not trigger any OPA policies."
    },
    "uid": "4570d0a2-8eb0-4a38-8053-b93c2cf0758f"
  }
}
```

**BreakGlass Allowed:**

```
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "allowed": true,
    "auditAnnotations": {
      "breakglass_authorized": "users, groups."
    },
    "uid": "4570d0a2-8eb0-4a38-8053-b93c2cf0758f"
  }
}
```

### Prometheus metrics:

In this section we are not going to focus on metrics that can be exported from Kubernetes cluster nodes or the different control and data plane components using the `/metrics` endpoints. Those are well known and there are many projects (e.g. Prometheus) that can help with that task. Instead, we will pay attention to the metrics exported by OPA itself to inform us about the performance of our admission controller at the application level.

OPA exposes an HTTP endpoint that can be used to collect performance metrics for all API calls. The Prometheus endpoint is enabled by default when you run OPA as a server and exports Go runtime metrics as well as HTTP request latency metrics for all handlers (e.g. `v1/data`). Metrics exposed by this endpoint are available here: [https://www.openpolicyagent.org/docs/latest/monitoring/#prometheus](https://www.openpolicyagent.org/docs/latest/monitoring/#prometheus).

In addition, there is another endpoint that could be enabled (e.g. via the CLI switch `--set="status.prometheus=true"`) to expose status metrics. The metric available in that endpoint can be found here: [https://www.openpolicyagent.org/docs/latest/monitoring/#status-metrics](https://www.openpolicyagent.org/docs/latest/monitoring/#status-metrics).

### Kube-apiserver metrics:

From a monitoring perspective, it is extremely useful to know if requests are being denied because of the policies or because the admission controller (i.e. kube-policies) is down and the failure policy is set to Fail. Fortunately, the kube-apiserver component exports a metric called `apiserver_admission_webhook_rejection_count` that contains internal attributes to let you know the reason for the rejection. More information about these metrics can be found here: [https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhook-metrics](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#admission-webhook-metrics). Below are some examples of rejection count metrics:

```
# HELP apiserver_admission_webhook_rejection_count [ALPHA] Admission webhook rejection count, identified by name and broken out for each admission type (validating or admit) and operation. Additional labels specify an error type (calling_webhook_error or apiserver_internal_error if an error occurred; no_error otherwise) and optionally a non-zero rejection code if the webhook rejects the request with an HTTP status code (honored by the apiserver when the code is greater or equal to 400). Codes greater than 600 are truncated to 600, to keep the metrics cardinality bounded.

# TYPE apiserver_admission_webhook_rejection_count counter

apiserver_admission_webhook_rejection_count{error_type="calling_webhook_error",name="always-timeout-webhook.example.com",operation="CREATE",rejection_code="0",type="validating"} 1
apiserver_admission_webhook_rejection_count{error_type="calling_webhook_error",name="invalid-admission-response-webhook.example.com",operation="CREATE",rejection_code="0",type="validating"} 1
apiserver_admission_webhook_rejection_count{error_type="no_error",name="deny-unwanted-configmap-data.example.com",operation="CREATE",rejection_code="400",type="validating"} 13
```

## Next steps

At Block, kube-policies has been running reliably on our production clusters for about a year now. While we’re proud of its success, we would love to work on improvements for how our developers interact with our service:

### Feedback loop

We primarily have two sets of clients who interact with kube-policies one way or another. Application owners need to create resources that are compliant with our policies with the help of our admission control mechanism. They typically rely on the events section of their resources or the direct response from the kube-apiserver. In contrast, platform operators are responsible for the infrastructure where these applications run. They require a more holistic approach to managing workloads, which is why they primarily rely on Kubernetes audit logs and whatever integration they might have with cloud providers (e.g. CloudWatch on AWS).

Currently, the first notification we give to the developers of an insecure pod configuration is when they try to deploy onto the cluster, get blocked, and receive our error message. Instead, we’d like to run our policies in CI, allowing us to point out configuration issues, and propose a remediation so the developers can easily adjust their config to meet our standards.

Another opportunity is a CLI tool to run policies over Pod specs. The CLI could return a list of policy violations or an updated version of the spec to meet our policies.

### Policy rollouts

Due to infrastructure constraints at the time and to ensure policy immutability, we rolled out kube-policies by embedding policies in the container image that implements its logic. This approach requires new deployments for any policy updates. While it can be expedited if necessary, since we impact production workloads, we put necessary emphasis on staged rollouts. We make sure to take sufficient time in-between environments for monitoring and testing. Adding exceptions to our policies also follows the same process which is inconvenient for our developers at best.

### Exception process

Adding an exception requires the developer to file a ticket with us, and then wait for us to create a PR to add the exception to our bundle. Then the PR must be approved and merged to build the newest image, and then be rolled out in stages to all environments. Each environment’s PR requires approvals from two different teams.

In the short term, we’d like to improve this process by providing a UI for developers to create or delete their exceptions which then automatically creates a PR to add the exception to our bundle. A long term goal is to remove the exceptions from the bundle.
