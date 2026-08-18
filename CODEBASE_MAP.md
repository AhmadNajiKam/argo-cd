# Argo CD codebase map for new contributors

This guide explains where things live, why they exist, and how important they are when you are learning the repository. It describes the current checkout, not every historical version of Argo CD.

> [!WARNING]
> Before turning any local change into a pull request, confirm that an open, approved GitHub issue explicitly requests the work. Read `AGENTS.md`, `CONTRIBUTING.md`, and `docs/developer-guide/code-contributions.md`. This repository rejects drive-by refactors and requires the pull request template, a semantic title, tests for behavior changes, and the prescribed build, generation, lint, test, and CLI checks.

## How to use this map

The repository has more than 5,000 tracked files. Listing thousands of fixture names without explaining their relationships would make a poor learning guide, so this map uses two levels of coverage:

1. Every top-level file and directory is mapped individually.
2. Inside each directory, every source package, functional folder, important hand-authored file, and recurring file family is mapped. Repeated generated files, snapshots, images, fixture manifests, and resource-customization cases are covered by explicit naming rules. Those rules account for every file in those families and explain why individual files exist.

Use `git ls-files <path>` when you need the exact members of a family described here. Do not start by reading generated code or every test fixture.

### Importance scale

| Rank | Meaning for a newcomer |
| --- | --- |
| **5 — essential** | Read early. This defines the product model, main runtime flow, or contribution workflow. |
| **4 — core** | Read when working on most backend, API, reconciliation, or UI features. |
| **3 — subsystem** | Important when your issue touches this specific component. |
| **2 — supporting** | Build, deployment, tests, documentation, or reusable infrastructure. Learn on demand. |
| **1 — specialized/generated** | Generated output, fixtures, assets, release automation, or narrow integrations. Usually do not edit directly. |

## The shortest useful reading path

If you know nothing about Argo CD, read in this order:

1. `README.md`, `docs/core_concepts.md`, and `docs/operator-manual/architecture.md` for the product vocabulary.
2. `pkg/apis/application/v1alpha1/types.go`, `app_project_types.go`, and `applicationset_types.go` for the Kubernetes API model.
3. `cmd/main.go` to see how one compiled Go program becomes the CLI and all server processes.
4. `controller/appcontroller.go`, `controller/state.go`, and `controller/sync.go` for the reconciliation loop.
5. `reposerver/server.go` and `reposerver/repository/repository.go` for Git/Helm/OCI access and manifest generation.
6. `gitops-engine/pkg/diff`, `gitops-engine/pkg/sync`, and `gitops-engine/pkg/cache` for comparison, apply/prune planning, and live Kubernetes state.
7. `server/server.go`, one `server/<service>/<service>.proto`, and its matching `<service>.go` for the API pattern.
8. `ui/src/app/app.tsx`, `ui/src/app/shared/services`, and the relevant feature directory if the issue is in the web UI.
9. `Makefile`, `Procfile`, and `docs/developer-guide/development-cycle.md` before building or testing.

## Runtime mental model

```text
Application / AppProject / ApplicationSet custom resources
                         |
                         v
      application controller <----> repo server <----> Git / Helm / OCI
                |                         |
                |                         +---- config-management plugin sidecars
                v
         GitOps Engine diff + sync planning
                |
                v
        destination Kubernetes cluster

Web UI / argocd CLI ---> API server ---> Kubernetes API, repo server, Redis/cache
                              |
                              +-- Dex / external OIDC for login

ApplicationSet controller ---> generates/updates Application resources
Notifications controller ----> watches Applications and sends notifications
Hydrator + commit server -----> optionally renders and commits hydrated manifests
```

Important boundaries:

- Kubernetes custom resources are the durable domain model. Start in `pkg/apis/application/v1alpha1`.
- The application controller owns reconciliation and operations; the repo server owns untrusted repository access and manifest rendering.
- `gitops-engine` is a nested Go module containing generic Kubernetes cache, diff, health, and sync machinery.
- The API server exposes gRPC services and a grpc-gateway REST surface used by the React UI.
- Redis is a performance/cache dependency, not the authoritative store for Applications.
- Most deployable process commands are selected by `cmd/main.go` based on the invoked binary name or `ARGOCD_BINARY_NAME`.

## Top-level directory map

| Rank | Path | Why it exists | Read/edit when… |
| --- | --- | --- | --- |
| **4** | `.github/` | GitHub issue/PR policy, CI workflows, dependency automation, security workflows, and triage automation. | CI fails, contribution metadata changes, or you need to understand required checks. |
| **1** | `.agents/`, `.codex/` | Local automation-agent metadata/instructions for tooling in this checkout. They are not Argo CD runtime code. | An agent workflow explicitly tells you to inspect them. |
| **5** | `applicationset/` | ApplicationSet reconciler, generators, templates, SCM integrations, progressive syncs, status, metrics, and examples. | The issue concerns generating Applications from lists, Git, clusters, SCMs, pull requests, plugins, matrix/merge generators, or rollout groups. |
| **2** | `assets/` | Go-embedded API-server assets: RBAC defaults/model, Swagger specification, and badge art. | Authorization defaults, API docs, or status badges change. |
| **5** | `cmd/` | The unified Go entrypoint and Cobra command wiring for every executable and the end-user CLI. | Adding flags, starting a component, changing CLI commands, or tracing dependency construction. |
| **3** | `cmpserver/` | Config Management Plugin sidecar RPC protocol, client, config loading, and server. | A custom manifest-generation plugin or sidecar protocol changes. |
| **3** | `commitserver/` | Optional source-hydration Git commit service and its RPC client, metrics, credentials, and repository handling. | Working on the source hydrator or committing rendered manifests back to Git. |
| **4** | `common/` | Small cross-component constants and build/version metadata. | A command name, shared label/annotation/config constant, or version output changes. |
| **5** | `controller/` | The main Application/AppProject reconciliation, comparison, health, sync, cluster cache, sharding, hydration, and metrics logic. | Almost any desired-vs-live behavior, sync, health, refresh, or application status issue. |
| **3** | `docs/` | User, operator, developer, security, upgrade, proposal, and generated CLI documentation plus images. | Behavior/configuration changes or the issue is documentation-specific. |
| **2** | `examples/` | Runnable/example dashboards, RBAC, known-hosts, and plugin configurations. | A user-facing example or integration needs demonstration. |
| **5** | `gitops-engine/` | Nested Go module implementing generic live-state cache, diff, resource health, reconciliation, sync waves/hooks, apply/prune, and utilities. | The bug is below Argo CD orchestration in Kubernetes comparison or synchronization. |
| **2** | `hack/` | Code generation, tool installation, manifest generation, release, testing, and maintenance programs/scripts. | A Make target delegates here or generated output must be refreshed. |
| **4** | `manifests/` | Kustomize sources, CRDs, RBAC, component workloads, install bundles, HA variants, and Tilt development resources. | Installation topology, permissions, deployment flags, services, or CRDs change. |
| **3** | `notification_controller/` | Argo CD-specific glue around the notifications engine. | Application watch or notification controller behavior changes. |
| **2** | `notifications_catalog/` | Built-in notification templates and triggers, plus their generated install bundle. | Default notification messages or trigger expressions change. |
| **1** | `overrides/` | MkDocs theme localization override. | Documentation site chrome needs adjustment. |
| **5** | `pkg/` | Public/domain Go packages: CRD types, generated Kubernetes clients/informers/listers, generated API clients, and rate limiting. | The API schema, Kubernetes object model, or clients change. |
| **1** | `renovate-presets/` | Shared Renovate dependency grouping/version policies and custom managers. | Dependency automation behavior changes. |
| **1** | `renovate.json` | Root Renovate configuration consuming the local presets. | Updating dependency-bot configuration. |
| **5** | `reposerver/` | Repository RPC service, Git/Helm/OCI fetching, manifest generation, caching, locking, signature metadata, and metrics. | Desired manifests, repository credentials, tool invocation, or revision resolution are involved. |
| **3** | `resource_customizations/` | Embedded Lua health checks, resource actions, discovery scripts, and YAML test cases for built-in and third-party Kubernetes kinds. | A resource appears with the wrong health/action behavior. |
| **5** | `server/` | API server composition and individual gRPC/REST services for applications, projects, clusters, repositories, sessions, settings, etc. | CLI/UI/API behavior, auth, RBAC enforcement, streaming, proxying, or CRUD changes. |
| **3** | `test/` | Cross-package fixtures, end-to-end suites, test repositories/registries, container harnesses, and remote-cluster tests. | Behavior crosses component boundaries or needs an e2e regression test. |
| **1** | `tools/` | Small developer-only programs; currently command-reference generation. | Generated CLI docs need changes. |
| **4** | `ui/` | React/TypeScript web application, frontend tests, styles/assets, build configuration, and embedded production bundle. | The web interface or its REST/SSE clients change. |
| **4** | `util/` | Reusable Argo CD infrastructure for Git, Helm, Kubernetes, cache, settings, auth, RBAC, sessions, webhooks, Lua, OCI, tracing, and more. | A core component delegates to a shared facility. Search callers before editing. |

## Top-level file map

| Rank | File | Purpose and editing guidance |
| --- | --- | --- |
| **5** | `AGENTS.md` | Mandatory automated-agent contribution rules: approved issue first, no drive-by work, semantic PR titles, complete template, tests/docs, and required checks. |
| **2** | `CHANGELOG.md` | Release history. Normally maintained through the release process, not casually edited. |
| **1** | `CLAUDE.md` | Instructions for another coding-assistant environment; not runtime code. |
| **3** | `CODEOWNERS` | Maps paths to GitHub review owners. Useful for finding domain maintainers. |
| **5** | `CONTRIBUTING.md` | Short contribution entrypoint linking to the full developer guide. |
| **3** | `Dockerfile` | Production multi-stage image: installs Helm/Kustomize/Git tooling, builds UI and Go binary, then creates component-name symlinks to the unified binary. |
| **2** | `Dockerfile.dev` | Lightweight development image layer around a prebuilt `argocd` binary. |
| **2** | `Dockerfile.tilt` | Debug-friendly Tilt image with development tools and Delve. |
| **2** | `Dockerfile.ui.tilt` | Tilt image/build path dedicated to live UI development. |
| **2** | `LICENSE` | Apache 2.0 project license. |
| **2** | `MAINTAINERS.md` | Current project maintainer list and governance-oriented contact information. |
| **5** | `Makefile` | Canonical build/test/codegen/lint/release command graph. Prefer its targets over inventing ad-hoc commands. Many non-`-local` targets use the project test-tools image. |
| **2** | `OWNERS` | Kubernetes-style repository ownership/approval metadata. Subdirectories may override it. |
| **4** | `Procfile` | Local multi-process topology used by `make start-local`/goreman and e2e development: controller, API, repo server, Redis, Dex, UI, ApplicationSet, notifications, CMP, commit server, and fixture servers. |
| **5** | `README.md` | Product overview and community/documentation entrypoint. |
| **2** | `SECURITY-INSIGHTS.yml` | Machine-readable project security posture metadata. |
| **3** | `SECURITY.md` | Vulnerability reporting and supported-version policy. Never report vulnerabilities in a public issue. |
| **2** | `SECURITY_CONTACTS` | Security response contacts. |
| **3** | `Tiltfile` | Kubernetes development environment: builds/debugs the unified binary, deploys `manifests/dev-tilt`, configures port forwards, and wires live updates. |
| **1** | `USERS.md` | Organizations publicly identifying as Argo CD users. |
| **3** | `VERSION` | Source version injected into builds and image tags by the Makefile. |
| **2** | `entrypoint.sh` | Container entrypoint wrapper; uses `tini` when running as PID 1 to reap child processes. |
| **5** | `go.mod` | Root Go module and dependency/version contract. It replaces the separately versioned `gitops-engine` module with the checked-in local directory. |
| **1** | `go.sum` | Root Go dependency checksums. Let Go tooling update it with intentional dependency changes. |
| **3** | `mkdocs.yml` | Complete documentation navigation and MkDocs Material configuration. New docs generally need a nav entry here. |
| **1** | `prepare.sh` | Historical migration helper used to move GitOps Engine into this repository. It is not a normal development setup script. |
| **1** | `sonar-project.properties` | Sonar static-analysis configuration. |
| **2** | `.codecov.yml` | Coverage upload/report behavior. |
| **2** | `.dockerignore` | Files omitted from Docker build contexts. |
| **1** | `.gitattributes` | Git path attributes such as generated-file/language handling. |
| **1** | `.gitignore` | Untracked build, editor, dependency, and local-runtime outputs. |
| **4** | `.golangci.yaml` | Go lint rules and exclusions used by lint targets/CI. |
| **2** | `.goreleaser.yaml` | CLI/release artifact packaging configuration. |
| **2** | `.mockery.yaml` | Root Mockery configuration for generated Go interfaces/mocks. |
| **2** | `.readthedocs.yaml` | Read the Docs build image, Python version, and docs installation settings. |
| **2** | `.snyk` | Snyk scan policy and ignores. |

## `cmd/`: executables and CLI

### Dispatch and shared wiring

| Rank | Path | Purpose |
| --- | --- | --- |
| **5** | `cmd/main.go` | Only top-level Go `main`. It inspects `argv[0]` or `ARGOCD_BINARY_NAME`, selects the matching Cobra root command, executes it, and falls back to CLI plugin handling. This explains why container symlinks all point to one binary. |
| **4** | `cmd/util/` | Shared dependency/config helpers for command startup: applications, ApplicationSets, clusters, projects, repositories, and common Kubernetes/client wiring. `*_test.go` files protect that wiring. |

### Deployable commands

Every directory below contains a `commands/` package. Its main production file creates flags, clients, caches, metrics/health endpoints, and the long-running server/controller. The adjacent tests verify flag defaults and construction.

| Rank | Directory | Process selected by `cmd/main.go` |
| --- | --- | --- |
| **5** | `cmd/argocd/commands/` | End-user `argocd` CLI root and command groups. |
| **5** | `cmd/argocd-server/commands/` | API server startup and all service dependencies. |
| **5** | `cmd/argocd-application-controller/commands/` | Application controller queues, cluster cache, repo/commit clients, and metrics startup. |
| **5** | `cmd/argocd-repo-server/commands/` | Repository server startup, parallelism, caches, TLS, and plugin discovery. |
| **4** | `cmd/argocd-applicationset-controller/commands/` | controller-runtime manager for ApplicationSets, webhooks, generators, and progressive syncs. |
| **3** | `cmd/argocd-notification/commands/` | Notifications controller command and troubleshooting subcommands. |
| **3** | `cmd/argocd-cmp-server/commands/` | Config Management Plugin sidecar process. |
| **3** | `cmd/argocd-commit-server/commands/` | Hydration commit service. |
| **3** | `cmd/argocd-dex/commands/` | Dex configuration generation and launch integration. |
| **3** | `cmd/argocd-k8s-auth/commands/` | Kubernetes exec-credential helper; `aws.go`, `gcp.go`, and `azure_*` implement cloud-provider login with build-tag variants. |
| **2** | `cmd/argocd-git-ask-pass/commands/` | Secure Git askpass helper used to supply credentials to Git subprocesses. |

### CLI file families

Within `cmd/argocd/commands/`, production filenames map directly to user command areas: `app*.go` (application CRUD/diff/sync/resources/actions), `applicationset.go`, `account.go`, `cert.go`, `cluster.go`, `gpg.go`, `login.go`/`logout.go`/`relogin.go`, `project*.go`, `repo.go`/`repocreds.go`, `context.go`, `configure.go`, `completion.go`, `plugin.go`, `tree.go`, `version.go`, and `root.go`. `admin/` holds direct Kubernetes/offline administrative operations; `headless/` supports noninteractive use; `initialize/` handles initialization; `utils/` contains prompt helpers. Each matching `*_test.go` tests the named command.

## `pkg/`: domain and generated clients

### `pkg/apis/application/v1alpha1/` — rank 5

This is the most important schema directory. Although the API remains named `v1alpha1`, it defines Argo CD's core Kubernetes objects and many internal wire/domain types.

| File/family | Purpose |
| --- | --- |
| `types.go` | Main `Application`, application spec/status, sources, sync policy/operation/result, health/sync status, resource status, and related types/methods. |
| `app_project_types.go` | `AppProject`, source/destination restrictions, roles, JWTs, sync windows, and policy helpers. |
| `applicationset_types.go` | `ApplicationSet`, generators, templates, strategies, rolling/progressive sync configuration, and status. |
| `repository_types.go` | Repository and credential-related domain types. |
| `application_annotations.go` | Well-known Application annotations. |
| `application_defaults.go` | Defaulting behavior. |
| `cluster_constants.go` | Cluster-secret labels/keys and related constants. |
| `source_integrity.go` | Source signature/integrity data. |
| `values.go` | Helm/Kustomize/source value helper types and serialization behavior. |
| `register.go`, parent `pkg/apis/application/register.go`, `doc.go` | Kubernetes scheme registration and API group metadata. |
| `generated.proto`, `generated.pb.go` | Protobuf representation and generated Go serialization. Edit the source types/generation inputs, not generated Go. |
| `openapi_generated.go` | Generated OpenAPI schema used for CRDs/API discovery. |
| `zz_generated.deepcopy.go` | Generated Kubernetes deep-copy methods. |
| `hack.go` | Code-generation compatibility hooks. |
| `*_test.go` | Unit tests for the correspondingly named schema/helpers. |

Any API struct change can cascade into protobuf, OpenAPI, deep copies, CRDs, clients, Swagger, docs, and manifests. Run the required code generation rather than hand-editing outputs.

### Other `pkg/` directories

| Rank | Path | Purpose |
| --- | --- | --- |
| **4** | `pkg/apiclient/apiclient.go` | Root client configuration/connectivity for the Argo CD API. |
| **3** | `pkg/apiclient/grpcproxy.go` | gRPC proxy/client support. |
| **1** | `pkg/apiclient/<service>/*.pb.go` | Generated gRPC messages/client/server interfaces from `server/<service>/<service>.proto`. |
| **1** | `pkg/apiclient/<service>/*.pb.gw.go` | Generated grpc-gateway REST adapters. |
| **2** | `pkg/apiclient/<service>/forwarder_overwrite.go` | Hand-written stream/proxy behavior that generated gateways cannot express. |
| **1** | `pkg/apiclient/**/mocks/` | Generated mocks for service clients/servers. |
| **1** | `pkg/client/clientset/` | Generated typed Kubernetes clients for Argo CD CRDs, with `fake/` clients for tests and `scheme/` registration. |
| **1** | `pkg/client/informers/` | Generated shared informer factories and per-resource informers. |
| **1** | `pkg/client/listers/` | Generated cache-backed listers for Applications, ApplicationSets, and AppProjects. |
| **3** | `pkg/ratelimiter/ratelimiter.go` | Configurable workqueue rate limiter used by the application controller. |
| **1** | `pkg/apis/api-rules/violation_exceptions.list` | Allow-list for intentional Kubernetes API lint exceptions. |

## `controller/`: Application reconciliation

| Rank | File/directory | Responsibility |
| --- | --- | --- |
| **5** | `appcontroller.go` | `ApplicationController`, informers, work queues, refresh/operation/hydration workers, status persistence, auto-sync decisions, deletion handling, and top-level reconciliation orchestration. |
| **5** | `state.go` | `AppStateManager`: asks repo server for desired objects, obtains live objects, normalizes/diffs them, calculates sync and health, builds resource status, and resolves revisions. |
| **5** | `sync.go` | Turns an Application operation into a GitOps Engine sync context, enforces project/windows/options, applies/prunes resources, runs hooks, updates operation state, and handles termination/retries. |
| **4** | `health.go` | Application-level health aggregation and health persistence rules. |
| **4** | `hook.go` | Argo CD resource-hook helpers and hook finalization/cleanup behavior. |
| **3** | `sort_delete.go` | Deterministic dependency-aware resource deletion ordering. |
| **3** | `sync_namespace.go` | Creation/management of destination namespaces during sync. |
| **3** | `clusterinfoupdater.go` | Keeps cluster connection/sharding information fresh. |
| **3** | `hydrator_dependencies.go` | Constructs dependencies for source hydration. |
| **3** | `hydrator/` | Hydration state machine/orchestration; `types/` defines queue/state types and `mocks/` supports tests. |
| **4** | `cache/` | Argo CD wrapper around live cluster state, resource ownership, tree information, and cache queries. |
| **3** | `sharding/` | Assigns destination clusters to controller replicas; `consistent/` contains consistent-hash implementation. |
| **3** | `metrics/` | Controller Prometheus metrics, cluster collector, and transport instrumentation. |
| **2** | `syncid/` | Generates/handles sync identifiers. |
| **1** | `testdata/` | Live/target YAML and schemas used by controller tests. Names identify the scenario. |
| **2** | `*_test.go`, `*_norace_test.go`, `*_informer_test.go` | Unit, informer, regression, concurrency, and build-mode tests for the same-named production area. |
| **2** | `OWNERS` | Review ownership for controller changes. |

For a reconcile bug, follow the path `appcontroller.go` → `state.go` for comparison/status or `sync.go` for mutation. If the surprising behavior is generic apply/diff/cache behavior, continue into `gitops-engine`.

## `reposerver/`: repository and manifest service

| Rank | File/directory | Responsibility |
| --- | --- | --- |
| **5** | `server.go` | Repo server construction, gRPC serving, health, TLS, lifecycle, and high-level dependencies. |
| **5** | `repository/repository.go` | Main RPC implementation: revision resolution, manifest generation, repo initialization, Helm/OCI/Git handling, metadata, and caching. |
| **4** | `repository/repository.proto` | Source definition of the repo-server RPC contract. Generated client code derives from it. |
| **3** | `repository/chart.go` | Helm chart/version detail handling. |
| **3** | `repository/lock.go` | Repository/revision concurrency locking so checkouts and generation do not corrupt each other. |
| **3** | `repository/types.go`, `utils.go` | Internal manifest-generation inputs/results and helpers. |
| **4** | `apiclient/clientset.go`, `repository.go` | Hand-written repo-server client connection and helpers. |
| **1** | `apiclient/repository.pb.go`, `apiclient/mocks/` | Generated RPC code and mocks. |
| **3** | `cache/` | Repo result/revision/manifest cache facade and mocks. |
| **3** | `metrics/` | Prometheus server and Git/OCI request instrumentation. |
| **2** | `gpgwatcher.go` | Watches trusted GPG key configuration used for signature verification. |
| **1** | `repository/testdata/` | Invalid/boundary manifest fixtures. |
| **2** | `*_test.go`, `OWNERS` | Matching unit tests and review ownership. |

Manifest tool wrappers live mostly under `util/git`, `util/helm`, `util/kustomize`, `util/oci`, `util/cmp`, and `util/lua`; the repo server composes them.

## `gitops-engine/`: diff, cache, health, and sync

This directory is a separately versioned nested Go module (`github.com/argoproj/argo-cd/gitops-engine/v3`). Run its tests as well as root-module tests when modifying it.

| Rank | Path | Purpose |
| --- | --- | --- |
| **5** | `pkg/cache/` | Watches Kubernetes resources, caches live state, discovers API resources, manages hierarchy/ownership, and emits updates. `mocks/` and fixtures support tests. |
| **5** | `pkg/diff/` | Desired-vs-live comparison, structured merge/server-side diff support, normalization, secret handling, and diff results. `internal/fieldmanager/` models managed-field behavior. |
| **5** | `pkg/sync/` | Reconciliation plan and execution: apply, prune, hooks, phases, waves, dependencies, tasks, results, and dry-run behavior. |
| **4** | `pkg/sync/common/` | Shared sync phases, result codes, annotations, and operation types. |
| **4** | `pkg/sync/hook/` | Hook classification/lifecycle; `hook/helm/` recognizes Helm hooks. |
| **4** | `pkg/sync/ignore/` | Determines resources that sync should skip. |
| **4** | `pkg/sync/resource/` | Resource operation helpers, annotations, and lifecycle rules. |
| **4** | `pkg/sync/syncwaves/` | Parses and orders sync waves. |
| **4** | `pkg/health/` | Built-in Kubernetes resource health evaluation and aggregation primitives. |
| **3** | `pkg/engine/` | Higher-level reconciliation engine facade. |
| **3** | `pkg/utils/kube/` | Kubernetes discovery, kubectl wrapper, apply/delete helpers, resource conversion, and testing/mocks/scheme helpers. |
| **2** | `pkg/utils/{io,json,text,tracing}/` | Narrow reusable helpers used by engine packages. |
| **1** | `internal/kubernetes_vendor/` | Selected Kubernetes implementation copied internally for behavior unavailable as a public API. Change cautiously and preserve provenance. |
| **1** | `agent/` | Example/experimental standalone agent and manifests demonstrating engine use. |
| **2** | `hack/update_static_schema.sh` | Refreshes static Kubernetes schema data. |
| **3** | `go.mod`, `go.sum`, `Makefile`, `.mockery.yaml` | Nested module dependency, build/test, and mock-generation configuration. |
| **2** | `README.md`, `LICENSE`, `OWNERS` | Module contract, licensing, and ownership. |
| **1** | Any `testdata/`, `mocks/`, `*_test.go` | Scenario fixtures, generated doubles, and matching tests. |

## `server/`: API server and services

`server/server.go` is rank 5: it composes HTTP, gRPC, grpc-gateway, authentication, RBAC interceptors, static UI serving, extensions, watchers, TLS, and each service implementation.

Each service directory follows a repeatable contract:

- `<name>.proto`: hand-authored API contract. HTTP annotations define REST routes.
- `<name>.go`: hand-authored implementation, validation, database/Kubernetes interaction, and RBAC checks.
- `<name>_test.go`: unit/regression tests.
- Generated `*.pb.go`/`*.pb.gw.go` live under `pkg/apiclient/<name>/`, not beside most server implementations.

| Rank | Service/folder | Responsibility |
| --- | --- | --- |
| **5** | `application/` | Application CRUD, validation, sync/rollback/terminate, diff/manifests, managed resources, resource tree, logs, exec terminal, WebSocket/SSE/watch streams, and deep informers. Largest API surface. |
| **4** | `project/` | AppProject CRUD, roles/tokens, validation, sync windows, and project-scoped policy. |
| **4** | `cluster/` | Destination-cluster registration, credentials, connection tests, rotation, and cluster metadata. |
| **4** | `repository/`, `repocreds/` | Repository and credential-template CRUD/validation, delegating repository checks/rendering to repo server. |
| **4** | `session/` | Login/token creation and rate limiting. Core auth path. |
| **4** | `settings/` | Public UI/client settings and auth configuration; `oidc/` discovers and adapts OIDC metadata. |
| **3** | `applicationset/` | API CRUD/watch/tree access for ApplicationSet resources. The controller logic remains under `applicationset/`. |
| **3** | `account/` | Account information, password update, and token operations. |
| **3** | `certificate/` | Repository TLS/SSH certificate management. |
| **3** | `events/` | Kubernetes event retrieval/watch API. |
| **3** | `notification/` | Notification listing/triggering API. |
| **3** | `gpgkey/` | GPG public-key management API. |
| **3** | `rbacpolicy/` | RBAC policy validation/checking API. |
| **3** | `extension/` | Reverse proxy for configured backend extensions; `mocks/` supports tests. |
| **2** | `badge/` | SVG application status badge HTTP endpoint and color logic. |
| **2** | `deeplinks/` | Evaluates configured resource/application deep links. |
| **2** | `logout/` | Logout handler/provider integration. |
| **2** | `version/` | Server/tool version API. |
| **2** | `broadcast/` | In-process fan-out for watch/event streams. |
| **2** | `cache/` | API-server-specific cached objects/results. |
| **2** | `metrics/` | API-server Prometheus metrics. |
| **2** | `rootpath_test.go`, `server_*_test.go` | Server composition, namespace filtering, and race/non-race tests. |

Read `docs/developer-guide/architecture/authz-authn.md` before altering authentication or RBAC. Service methods, not only global middleware, are responsible for authorization decisions.

## `applicationset/`: generating Applications

| Rank | Path | Purpose |
| --- | --- | --- |
| **5** | `controllers/applicationset_controller.go` | Main reconciler: evaluates generators/templates, validates Applications, creates/updates/deletes them according to policy, and writes status. |
| **4** | `controllers/clustereventhandler.go` | Maps cluster-secret changes to affected ApplicationSets and reconciliation. |
| **4** | `controllers/template/` | Renders Application templates, including Go-template behavior. |
| **4** | `controllers/progressive_sync_dependencies.go` | Dependency wiring for progressive-sync evaluation. |
| **5** | `generators/` | One file per generator (`list`, `cluster`, `git`, `matrix`, `merge`, `pull_request`, `scm_provider`, `plugin`, duck-type), plus processing/interpolation/interfaces/utilities. Matching tests describe edge cases. |
| **4** | `progressivesync/` | Rolling/progressive grouping, status evaluation, gating, validation, and regression tests. |
| **4** | `services/pull_request/` | Provider-neutral pull-request enumeration plus provider implementations/fixtures. |
| **4** | `services/scm_provider/` | SCM repository discovery implementations, Azure DevOps Git client, mocks, and test data. |
| **3** | `services/plugin/` | Calls external generator plugins. |
| **3** | `services/github_app_auth/`, `services/internal/github_app/` | GitHub App authentication helpers. |
| **3** | `services/internal/http/` | Hardened HTTP helpers shared by provider integrations. |
| **3** | `services/repo_service.go` | Repo-server adapter used by Git-backed generators. |
| **3** | `services/github_metrics.go` | GitHub API metrics. |
| **3** | `webhook/` | SCM webhook parsing/handling and payload fixtures. |
| **3** | `status/` | ApplicationSet resource/status aggregation. |
| **3** | `utils/` | Kubernetes clients/listers, selectors, policy, create-or-update, map and template functions; `mocks/` supports tests. |
| **2** | `metrics/` | Controller metrics and fake metrics for tests. |
| **2** | `examples/` | Generator- and strategy-specific manifests/apps. Each directory name states the feature demonstrated; nested guestbook/add-on files are sample desired state, not controller source. |

## `cmpserver/`, `commitserver/`, and notifications

### Config Management Plugins (`cmpserver/`) — rank 3

- `server.go` starts the plugin RPC server.
- `plugin/plugin.proto` is the RPC source contract; `apiclient/plugin.pb.go` is generated from it.
- `plugin/plugin.go` implements manifest generation, discovery, parameters, and streaming.
- `plugin/config.go` loads plugin configuration.
- `plugin/plugin_unix.go` and `plugin_windows.go` isolate OS-specific process/socket behavior.
- `apiclient/clientset.go` connects repo server to plugin sidecars.
- `plugin/testdata/` and `*_test.go` cover configurations and protocol behavior.

### Hydration commit service (`commitserver/`) — rank 3

- `server.go` starts the gRPC service.
- `commit/commit.proto` is its API; `apiclient/commit.pb.go` is generated.
- `commit/commit.go` implements repository initialization, commit/note operations, and RPC behavior.
- `commit/hydratorhelper.go` translates hydration requests/state.
- `commit/credentialtypehelper.go` selects supported credentials.
- `commit/repo_client_factory.go` creates Git clients.
- `apiclient/clientset.go` is the client facade; `mocks/` contains generated doubles.
- `metrics/` instruments Git operations and service behavior.
- `*_test.go` files cover the named behavior, including a race regression.

### Notifications

- `notification_controller/controller/controller.go` is the Argo CD controller adapter that watches Applications and feeds the reusable notifications engine; its test mirrors it.
- `notifications_catalog/templates/*.yaml` are built-in outbound message definitions, one per event.
- `notifications_catalog/triggers/*.yaml` are expressions deciding when each event fires.
- `notifications_catalog/install.yaml` is the combined generated/default install catalog. Change source template/trigger files and regenerate when required.

## `util/`: shared infrastructure directory-by-directory

Do not read `util/` linearly. Enter it from a caller or issue. Production files generally have same-named `*_test.go`; `mocks/` and `testdata/` are test support.

| Rank | Directory | Responsibility |
| --- | --- | --- |
| **4** | `util/app/discovery/` | Detects manifest source type/tooling and discoverable files. |
| **3** | `util/app/log/` | Application-scoped structured logging fields. |
| **3** | `util/app/path/` | Validates/resolves repository application paths. |
| **4** | `util/argo/` | Core Argo helpers, audit logging, resource tracking labels/annotations, and shared types. |
| **4** | `util/argo/diff/` | Argo-specific diff configuration layered over GitOps Engine: ignore rules, normalizers, server-side diff, and known-type handling. |
| **4** | `util/argo/managedfields/` | Managed-fields normalization/ignore behavior. |
| **4** | `util/argo/normalizers/` | Applies ignore-difference and JQ normalizers before diffing. |
| **3** | `util/askpass/` | Local credential RPC used by Git askpass; `.proto` is source and `.pb.go` generated. |
| **2** | `util/assets/` | Access to Go-embedded static assets. |
| **2** | `util/buffered_context/` | Context wrapper that buffers/cancels work safely. |
| **4** | `util/cache/` | Generic cache interfaces plus in-memory, Redis, and two-level clients/hooks/mocks. |
| **4** | `util/cache/appstate/` | Redis-backed application state, managed-resource, tree, and manifest cache. |
| **3** | `util/cert/` | Certificate parsing/fingerprinting helpers. |
| **3** | `util/cli/` | Common Cobra/logging/signals/client CLI setup. |
| **3** | `util/clusterauth/` | Builds Kubernetes REST configuration from Argo CD cluster credentials. |
| **3** | `util/cmp/` | Streams manifest data between repo server and plugin server. |
| **2** | `util/collections/` | Small map/collection utilities. |
| **3** | `util/config/` | Reads configuration from environment/files with typed conversion. |
| **3** | `util/crypto/` | Encryption/hash/key helpers for sensitive configuration. |
| **4** | `util/db/` | `ArgoDB` abstraction over Kubernetes Secrets/ConfigMaps for repositories, repo credentials, clusters, certificates, GPG keys, and related CRUD. |
| **3** | `util/dex/` | Generates/manages Dex configuration and execution. |
| **2** | `util/env/` | Environment-variable parsing helpers. |
| **2** | `util/errors/` | Shared typed/error helpers, including credential errors. |
| **4** | `util/exec/` | Hardened subprocess execution, cancellation, output, and environment handling. Security-sensitive because repo tools run here. |
| **4** | `util/git/` | Git client, clone/fetch/revision logic, credentials, SSH, GPG verification, and compatibility workarounds. |
| **3** | `util/github_app/` | Lists/accesses repositories through GitHub App credentials. |
| **2** | `util/glob/` | Glob matching/list helpers used by repository and generator logic. |
| **4** | `util/grpc/` | gRPC server/client configuration, interceptors, logging, JSON, error translation, sanitization, and user-agent handling. |
| **2** | `util/guard/` | Panic/recovery or lifecycle guard helpers. |
| **2** | `util/hash/` | Stable hashing helpers. |
| **2** | `util/healthz/` | HTTP health/readiness endpoint utilities. |
| **4** | `util/helm/` | Helm command/client, credentials, index parsing, chart rendering, and test fixtures/mocks. |
| **2** | `util/http/` | Shared HTTP client/server helpers. |
| **3** | `util/hydrator/` | Source-hydration rendering/template helpers used by controller and commit service. |
| **2** | `util/io/` | Readers, closers, filesystem composition, and path utilities; subfolders split file/path implementations and mocks. |
| **3** | `util/jwt/` | JWT claim/token utilities. |
| **4** | `util/kube/` | Kubernetes REST clients, kubectl wrapper, retries, port-forwarding, object helpers, and fixtures. |
| **4** | `util/kustomize/` | Kustomize command/render wrapper and fixtures. |
| **3** | `util/localconfig/` | Reads/writes CLI config and enforces OS-specific file permissions. |
| **2** | `util/log/` | Logrus configuration and Kubernetes klog bridge. |
| **4** | `util/lua/` | Sandboxed Lua execution for resource health, actions, discovery, impacted resources, and script caching. |
| **3** | `util/manifeststream/` | Efficient manifest streaming/serialization. |
| **2** | `util/metrics/` | Common Prometheus metric helpers; `kubectl/` instruments kubectl calls. |
| **3** | `util/notification/argocd/` | Argo CD notification context/expression helpers and mocks. |
| **3** | `util/notification/expression/` | Expression functions; subpackages expose repo, shared, string, and time functions. |
| **3** | `util/notification/k8s/` | Kubernetes-backed notification configuration/subscriptions. |
| **3** | `util/notification/settings/` | Notification settings parsing. |
| **4** | `util/oci/` | OCI registry client/auth/pull/metadata and mocks. |
| **4** | `util/oidc/` | OIDC providers, Azure variations, templates, discovery, and login helpers. |
| **3** | `util/password/` | Password hashing/verification policy. |
| **2** | `util/profile/` | Runtime profiling server/setup. |
| **3** | `util/proxy/` | Reverse-proxy configuration/helpers. |
| **1** | `util/rand/`, `util/regex/`, `util/resource/`, `util/stats/`, `util/templates/`, `util/text/`, `util/text/label/` | Small random, regex, revision/resource, statistics, template normalization, and label/text helpers. |
| **4** | `util/rbac/` | Casbin RBAC enforcer, policy loading, informer updates, and tests. |
| **4** | `util/security/` | Application namespace constraints, JWT/RBAC checks, and path-traversal defenses. Security-sensitive. |
| **4** | `util/session/` | Session manager, auth-provider integration, token verification/creation, OIDC state, and tests. |
| **4** | `util/settings/` | Watches/parses Argo CD ConfigMaps/Secrets, accounts, cluster informer settings, resource filters, and impersonation. |
| **3** | `util/sourceintegrity/` | Source integrity and GPG verification. |
| **2** | `util/swagger/` | Serves/rewrites embedded Swagger data. |
| **1** | `util/test/` | Common test helpers. |
| **2** | `util/tgzstream/` | Streaming tar-gzip packaging. |
| **3** | `util/tls/` | TLS configuration/certificate helpers and fixtures. |
| **2** | `util/trace/` | OpenTelemetry tracing setup. |
| **2** | `util/versions/` | Semantic version/tag parsing and compatibility helpers. |
| **3** | `util/webhook/` | Git provider, registry, GHCR, and Harbor webhook parsing/matching. |
| **4** | `util/workloadidentity/` | Cloud workload identity credentials with CGO/non-CGO variants and mocks. |
| **2** | `util/util.go` | Miscellaneous tiny cross-package helpers; avoid growing this catch-all without a good reason. |

## `ui/`: React/TypeScript web application

| Rank | Path | Purpose |
| --- | --- | --- |
| **4** | `ui/package.json` | pnpm scripts and frontend dependency contract. Use its `start`, `build`, `test`, `lint`, and formatting scripts. |
| **1** | `ui/pnpm-lock.yaml` | Exact dependency lock; regenerated by pnpm for intentional dependency changes. |
| **4** | `ui/src/app/index.tsx` | Browser entrypoint that mounts the React application. |
| **5** | `ui/src/app/app.tsx` | Top-level routes, navigation, auth/session bootstrap, global managers, extension registration, layout, and error handling. |
| **4** | `ui/src/app/index.html` | HTML shell/base path used by the API server and webpack. |
| **4** | `ui/src/app/webpack.config.js` | Development/production bundling, loaders, proxy, and asset output. |
| **3** | `ui/src/app/tsconfig.json`, root `ui/tsconfig.json` symlink | TypeScript compiler/editor configuration. |
| **4** | `ui/src/app/applications/` | Main Applications/ApplicationSets UX: list, detail tree, sync, diff, logs, terminal, events, resource info, history, parameters, retry, filters, and status. Each component folder contains its TSX/SCSS/tests/snapshots. |
| **3** | `ui/src/app/resources/` | Cross-application managed-resources view. |
| **4** | `ui/src/app/settings/` | Repositories, credentials, clusters, projects, accounts, certificates, GPG keys, appearance, roles, policies, and sync-window screens. |
| **3** | `ui/src/app/login/`, `user-info/`, `help/` | Login, current-user, and help/version feature areas. |
| **4** | `ui/src/app/shared/services/` | REST/SSE clients. `requests.ts` is the base HTTP/event-source layer; each `<domain>-service.ts` maps UI operations to server endpoints. |
| **4** | `ui/src/app/shared/models.ts` | Frontend domain/interface definitions corresponding to API objects. |
| **4** | `ui/src/app/shared/context.ts` | React context and shared provider state. |
| **3** | `ui/src/app/shared/components/` | Reusable layout, forms, editor, status, events, paging, search, badges, and other UI primitives. |
| **3** | `ui/src/app/shared/hooks/` | Reusable React hooks, query and list sorting. |
| **2** | `ui/src/app/shared/config.scss`, feature `*.scss` | Global tokens/config and feature styles. |
| **3** | `ui/src/app/sidebar/`, `ui-banner/` | Global navigation and configured banner. |
| **1** | `ui/src/assets/` | Fonts, logos, built-in Kubernetes resource icons, integration icons, and decorative images. Generated icon indexes may be refreshed through `hack/generate-icons-typescript.sh`. |
| **1** | `ui/dist/app/` | Built frontend embedded into the Go binary by `ui/embed.go`. Treat hashed JS/CSS/maps as generated build output. |
| **2** | `ui/embed.go` | Go embed bridge for the built UI bundle. |
| **2** | `ui/scripts/build_docker.sh` | Frontend/container build helper. |
| **2** | `ui/{babel.config.js,eslint.config.mjs,jest.config.js,jest.setup.js,.babelrc,.prettierrc,.jshintrc}` | Transpile, lint, unit-test, setup, and formatting configuration. |
| **1** | `ui/__mocks__/`, `**/__snapshots__/`, `*.test.ts`, `*.test.tsx` | Jest asset mocks, snapshot baselines, and tests paired with components/services/helpers. Update snapshots only after reviewing semantic changes. |
| **1** | `ui/.nvmrc`, `.dockerignore`, `.gitignore`, `osv-scanner.toml`, `LICENSE`, `OWNERS`, `README.md` | Node version, packaging ignores, dependency scanning, licensing/ownership, and UI-local setup. |

## `manifests/`: installation topology

| Rank | Path/family | Purpose |
| --- | --- | --- |
| **4** | `base/` | Canonical Kustomize resources grouped by component: controller, ApplicationSet, commit server, config, Dex, notifications, Redis, repo server, API server, and controller deployment/RBAC variants. |
| **4** | `base/config/` | Default Argo CD ConfigMaps/Secret placeholders: general config, command parameters, RBAC, GPG, SSH known hosts, TLS certificates. |
| **4** | `cluster-rbac/` | ClusterRoles and bindings for API server/controllers. Security-sensitive. |
| **5** | `crds/` | Generated installable CRDs for Application, ApplicationSet, and AppProject plus Kustomization. Source schema is under `pkg/apis/application/v1alpha1`. |
| **3** | `cluster-install/`, `namespace-install/`, `core-install/` | Kustomize compositions for cluster-wide, namespace-scoped, and core-only installations. |
| **3** | Matching `*-with-hydrator/` | Variants enabling commit server/source hydration. |
| **3** | `ha/` | High-availability overlays and pre-rendered install bundles. |
| **2** | `dev-tilt/` | Namespace, UI resources, and overlay used by `Tiltfile`. |
| **2** | `addons/` | Optional installation additions and guidance. |
| **1** | Top-level `install*.yaml`, `namespace-install*.yaml`, `core-install*.yaml`, and `ha/*.yaml` | Generated, flattened release artifacts consumed directly with `kubectl`. Modify Kustomize sources and regenerate. |
| **2** | Every `kustomization.yaml` | Declares the resources/patches composing that directory's variant. |
| **2** | `README.md`, `.gitignore` | Explains bundle choices and ignores local Kustomize output. |

Manifest filenames are descriptive: `*-deployment.yaml`, `*-statefulset.yaml`, `*-service.yaml`, `*-sa.yaml`, `*-role*.yaml`, `*-network-policy.yaml`, and `*-metrics*.yaml` define exactly that Kubernetes object for the named component.

## `resource_customizations/`: health and action scripts

This large tree is systematic:

```text
resource_customizations/<api-group>/<Kind>/
  health.lua             # returns health status/message for that unstructured object
  health_test.yaml       # table of fixture file -> expected health/message
  actions.lua            # optional collection of actions
  actions/<action>/action.lua
  action.lua             # action implementation in older/single-action layouts
  action_test.yaml       # action inputs and expected outputs
  discovery.lua          # decides whether an action is available/enabled
  testdata/*.yaml        # Kubernetes objects exercising health/action states
```

The API-group folder (`apps`, `batch`, `cert-manager.io`, `argoproj.io`, and many third-party groups) scopes the customization. The Kind folder selects a resource. `_` is a wildcard kind. Fixture names such as `healthy.yaml`, `degraded.yaml`, `progressing.yaml`, `suspended.yaml`, or a feature-specific name describe the state under test.

- `resource_customizations/embed.go` embeds the entire tree for runtime Lua lookup.
- Kubernetes built-ins and every third-party group/kind directory have rank 1 globally, rank 4 when your issue is about that exact resource.
- Never “fix” only a fixture. Change the Lua behavior and add/update cases in the corresponding test manifest.
- Health execution/sandbox code is in `util/lua`; application health aggregation is in `controller/health.go` and `gitops-engine/pkg/health`.

## `docs/`, examples, and generated documentation

| Rank | Path | Purpose |
| --- | --- | --- |
| **5** | `docs/core_concepts.md`, `understand_the_basics.md`, `getting_started.md` | Best product/domain introduction. |
| **5** | `docs/operator-manual/architecture.md` | Deployment component overview. |
| **4** | `docs/developer-guide/architecture/` | Component dependency and authentication/authorization design. |
| **5** | `docs/developer-guide/{development-environment,development-cycle,code-contributions,running-locally,test-e2e}.md` | Contributor setup, workflow, local runtime, and test instructions. |
| **3** | `docs/user-guide/` | End-user behavior organized by source type, sync/diff feature, project, resource, CLI, and Application specification. Update with user-visible features. |
| **3** | `docs/operator-manual/` | Installation/configuration/security/HA/auth/notifications/ApplicationSet/operations material. |
| **2** | `docs/proposals/` | Accepted/proposed designs and historical rationale. Useful before changing a feature's architecture; not always current implementation truth. |
| **1** | `docs/user-guide/commands/`, `docs/operator-manual/server-commands/` | Generated CLI and server-command reference. Change Cobra definitions and regenerate. |
| **1** | Paired `*-yaml.md`/`.yaml` configuration reference files | Generated or synchronized examples/reference material for Argo CD ConfigMaps and Secrets. Check generators before editing. |
| **1** | `docs/assets/` | Screenshots, diagrams, CSS, JavaScript, logos, GIFs, and sample asset files referenced by docs. Filename states the page/topic. |
| **1** | `docs/snyk/` | Generated Snyk reports by release/version. |
| **2** | Root docs such as `faq.md`, `roadmap.md`, `security_considerations.md`, `SUPPORT.md`, `CONTRIBUTING.md` | Cross-cutting reference and docs-specific contributor guidance. |
| **2** | `docs/requirements.txt` | Python dependencies for building MkDocs. |
| **2** | `examples/k8s-rbac/` | Example Kubernetes RBAC policies by API operation. |
| **2** | `examples/known-hosts/` | Kustomize example for mounting custom SSH known hosts. |
| **2** | `examples/plugins/` | Config-management-plugin examples, including Helm. |
| **1** | `examples/dashboard*.json` | Importable monitoring dashboard definitions. |
| **2** | `applicationset/examples/` | ApplicationSet generator/strategy examples described earlier. |

`mkdocs.yml` is the authoritative navigation map for documentation files. A documentation filename generally maps one-to-one to the user/operator topic named by that file.

## `test/`: cross-component verification

| Rank | Path | Purpose |
| --- | --- | --- |
| **4** | `test/e2e/*_test.go` | Full behavior suites. Filenames map to features: applications, sync, diff, repos, Helm/Kustomize/OCI, projects, clusters, ApplicationSet, hydration, notifications, sharding, auth, etc. |
| **4** | `test/e2e/fixture/` | Fluent e2e DSL, Given/When/Then helpers, API/CLI assertions, environment setup, and cleanup. Read before adding an e2e test. |
| **1** | `test/e2e/testdata*`, `test/manifests/` | Git repository and Kubernetes manifest fixtures used by e2e scenarios. Directory/file names state scenario/tool. |
| **3** | `test/fixture/` | Shared local integration fixtures: certificates, GPG, logging, repository paths, revision metadata, test repos, Git/Helm/OCI server setup. |
| **2** | `test/container/` | Containerized test-tools environment, Procfile, entrypoint, and cleanup/reaper. |
| **3** | `test/remote/` | Harness and RBAC/manifests for e2e tests against a remote destination cluster. |
| **2** | `test/cmp/` | Config Management Plugin fixture configuration/socket location. |
| **1** | `test/certificates/`, `test/testdata/` | Certificate and static data fixtures. |
| **3** | `test/manifests_test.go` | Verifies generated/committed install manifests. |
| **2** | `test/testutil.go`, `test/testdata.go` | Shared helpers and fixture access. |

Across the whole repository:

- `*_test.go` belongs to the same package/behavior as the filename without `_test`.
- `*_norace_test.go` is excluded or treated specially in race builds.
- `mocks/` usually contains generated interfaces and should be regenerated.
- `testdata/` is deliberately ignored by Go package discovery and stores fixtures.
- `__snapshots__/` is Jest-rendered output to review, not primary implementation.

## `hack/`, build, generation, and release tooling

| Rank | Path/file | Purpose |
| --- | --- | --- |
| **4** | `hack/update-codegen.sh` | Regenerates Kubernetes clients/informers/listers/deep copies. |
| **4** | `hack/generate-proto.sh` | Regenerates protobuf and grpc-gateway Go code. |
| **4** | `hack/update-openapi.sh` | Regenerates OpenAPI/Swagger artifacts. |
| **4** | `hack/update-manifests.sh`, `gen-crd-spec/` | Rebuilds CRDs/install manifests from source API and Kustomize inputs. |
| **3** | `hack/generate-mock.sh` | Regenerates Mockery output. |
| **3** | `hack/gen-docs/`, `tools/cmd-docs/main.go` | Generates CLI/server command docs. |
| **3** | `hack/gen-catalog/` | Generates notification catalog install/docs artifacts. |
| **3** | `hack/generate-actions-list.sh` | Generates resource action documentation/listing. |
| **3** | `hack/generate-icons-typescript.sh` | Generates frontend resource-icon TypeScript mappings. |
| **3** | `hack/gen-resources/` | Resource generation CLI/program, generators, examples, and utilities. |
| **3** | `hack/install.sh`, `hack/installers/`, `hack/tool-versions.sh` | Reproducibly installs pinned Helm, Kustomize, protoc, lint, codegen, OCI, Git LFS, and test tools. `checksums/` verifies downloads. |
| **3** | `hack/test.sh` | Test orchestration used by Make targets/CI. |
| **2** | `hack/Dockerfile.dev-tools`, `.dockerignore` | Reproducible test/developer tools image. |
| **2** | `hack/dev-mounter/` | Local program mirroring ConfigMap data into development mount paths. |
| **2** | `hack/k8s/`, `hack/known_types/` | Kubernetes helper/schema and known-type generation inputs. |
| **2** | `hack/goreman-start.sh`, `start-redis-with-password.sh` | Starts the local Procfile stack and Redis dependency. |
| **2** | `hack/git-ask-pass.sh`, `git-verify-wrapper.sh`, `gpg-wrapper.sh` | Git/GPG process integration wrappers used in runtime images. |
| **2** | `hack/admonitions-to-alerts.sh` | Converts documentation admonitions for another output format. |
| **2** | `hack/generate-release-notes.sh`, `get-previous-release/`, `trigger-release.sh`, `bump-major-version.sh` | Release-note, version, and release automation. |
| **2** | `hack/generate-ui-pnpm-sbom.sh`, `snyk-*.sh`, `snyk-report.sh` | Software bill of materials and vulnerability scanning/reporting. |
| **2** | `hack/update-kubernetes-version.sh`, `update-supported-versions.sh`, `update-ssh-known-hosts.sh` | Focused dependency/support/config maintenance. |
| **1** | `hack/migrate-gitops-engine/`, root `prepare.sh` | Historical/narrow GitOps Engine migration machinery. |
| **1** | `hack/boilerplate.go.txt`, `custom-boilerplate.go.txt`, `tools.go` | Generated-file headers and tool dependency anchors. |

If a file begins with `Code generated ... DO NOT EDIT`, find the owning Make/hack generator. `make codegen-local` aggregates most local generators; API changes require the prescribed full `make codegen` check.

## `.github/`, dependency, and repository administration

| Rank | Path | Purpose |
| --- | --- | --- |
| **5** | `.github/pull_request_template.md` | Required PR sections; do not delete or leave blank. |
| **4** | `.github/ISSUE_TEMPLATE/` | Bug, enhancement, release, security-log, and developer-tool issue forms/config. An approved issue is required before PR creation. |
| **3** | `.github/workflows/ci-build.yaml` | Main build/test/lint CI orchestration. |
| **2** | `.github/workflows/image*.yaml`, `release.yaml`, `init-release.yaml`, `bump-major-version.yaml` | Image reuse/build and release automation. |
| **2** | `.github/workflows/{codeql,scorecard,zizmor,update-snyk}.y*ml` | Security analysis and report updates. |
| **2** | `.github/workflows/{pr-title-check,cherry-pick,cherry-pick-single,renovate,stale}.y*ml` | PR policy, backport, dependency, and stale-item automation. |
| **1** | `.github/workflows/issue-triage.md`, `issue-triage.lock.yml`, `aw.json`, `.github/aw/` | Agentic workflow source, compiled lock/config, and action pinning for issue triage. |
| **2** | `.github/dependabot.yml`, `no-response.yml`, `stale.yml`, `pr-title-checker-config.json`, `zizmor.yml` | Bot/policy configuration. |
| **2** | `.github/configs/renovate-config.js`, `renovate.json`, `renovate-presets/` | Root and reusable Renovate policies, custom managers, version fixes, and dependency groups. |
| **2** | `.github/workflows/README.md` | Workflow maintenance guidance. |

## Small support directories and files

### `assets/`

- `embed.go` embeds all sibling assets into Go.
- `builtin-policy.csv` is default Casbin RBAC policy.
- `model.conf` is the Casbin authorization model.
- `swagger.json` is generated API documentation consumed by Swagger handlers.
- `badge.svg` is the status badge template/artwork.

The RBAC files are rank 4 because defaults affect authorization; generated Swagger and badge art are rank 1–2.

### `common/`

- `common.go` defines shared command names, labels/annotations, config names/keys, environment constants, and other cross-component identifiers.
- `version.go` stores build metadata and formats version information.
- Their `*_test.go` files lock the corresponding behavior.

### `examples/`, `overrides/`, `tools/`, and Renovate

- `overrides/partials/language/en-custom.html` overrides the MkDocs theme's English “Table of Contents” label.
- `tools/cmd-docs/main.go` generates Cobra command-reference Markdown.
- Each `renovate-presets/*.json5` groups a dependency domain (production binaries, dev tools, Dex, docs, Redis); `custom-managers/` extracts nonstandard version declarations and `fix/` supplies compatibility/version overrides.

## Recurring file conventions: how every remaining file maps

| Pattern | Meaning | Edit policy |
| --- | --- | --- |
| `*.go` | Hand-authored Go unless it carries a generated header. Package name and sibling files define its subsystem. | Format with `gofmt`; add/update matching tests. |
| `*_test.go` | Go tests for the same package/feature named before `_test`. | Preferred place for regressions. |
| `*_cgo.go`, `*_no_cgo.go`, `*_unix.go`, `*_windows.go` | Build/platform-specific implementation. | Preserve behavior across variants or document why it differs. |
| `*.proto` | Source gRPC/protobuf API contract. | Edit this, then regenerate `*.pb.go` and gateway output. |
| `*.pb.go`, `*.pb.gw.go`, `zz_generated.*`, `openapi_generated.go`, generated clientsets/informers/listers/mocks | Generated Go. | Do not hand-edit; change sources/generator and run codegen. |
| `kustomization.yaml` | Kustomize composition/overlay. | Edit with its referenced resources/patches; regenerate flat bundles. |
| Install/CRD aggregate `*.yaml` | Generated deployment/API artifact. | Usually regenerate from source types/Kustomize. |
| `testdata/**/*.yaml|json`, `fixtures/` | Named input/expected result for the nearest test package. | Treat filename and parent test as its mapping. |
| `resource_customizations/**/*.lua` | Sandboxed health/action/discovery code for the parent API group/kind/action. | Pair behavior changes with YAML cases. |
| `resource_customizations/**/*.yaml` | Input/expected fixture for the nearest Lua customization. | Do not read unless working on that kind. |
| `docs/**/*.md` | Documentation for the topic expressed by its relative path and MkDocs nav label. | Update with feature changes; include fenced-code language identifiers. |
| `docs/assets/*`, `ui/src/assets/*` | Visual/static asset referenced by the nearest docs/UI feature or generated icon map. | Binary assets are low-value reading. |
| `ui/**/*.tsx` | React component or test; folder name is the feature/component. | Keep service/domain logic out of purely presentational code where practical. |
| `ui/**/*.ts` | Service, model, helper, hook, configuration, or test named by file. | Follow existing adjacent patterns. |
| `ui/**/*.scss` | Styling scoped to the sibling feature/component. | Run UI lint/tests/build. |
| `ui/dist/**/*` | Hashed generated production bundle and source maps. | Build, do not hand-edit. |
| `OWNERS`, `CODEOWNERS` | Review/approval responsibility. | Administrative, not runtime. |
| `README.md` in a subdirectory | Local setup/contract for that subtree. | Read before changing the subtree. |

## “Where should I make this change?” lookup

| Symptom/feature | Start here | Common next stop |
| --- | --- | --- |
| Application says OutOfSync unexpectedly | `controller/state.go` | `util/argo/diff`, `gitops-engine/pkg/diff` |
| Sync applies/prunes in the wrong order | `controller/sync.go` | `gitops-engine/pkg/sync`, `syncwaves`, `resource`, `hook` |
| Resource health is wrong | `resource_customizations/<group>/<Kind>` | `util/lua`, `gitops-engine/pkg/health`, `controller/health.go` |
| Git/Helm/Kustomize/OCI rendering is wrong | `reposerver/repository/repository.go` | matching `util/{git,helm,kustomize,oci}` package |
| Application status does not refresh | `controller/appcontroller.go` | `controller/cache`, `state.go`, `util/cache/appstate` |
| API endpoint/CLI request is wrong | `server/<service>/<service>.proto` and `.go` | `pkg/apiclient/<service>`, `cmd/argocd/commands` |
| Login/token/RBAC problem | `util/session`, `util/rbac`, `util/security` | `server/session`, `server/server.go`, auth architecture docs |
| ApplicationSet output is wrong | `applicationset/controllers/applicationset_controller.go` | corresponding `applicationset/generators/*.go` or service |
| UI data is wrong but API is right | relevant `ui/src/app/...` component | `ui/src/app/shared/services`, `models.ts` |
| Cluster connection/credentials fail | `util/clusterauth`, `util/db/cluster.go` | `server/cluster`, `util/kube` |
| Webhook does not refresh | `util/webhook` | repo server or ApplicationSet `webhook/` depending on event |
| Installation/RBAC is wrong | `manifests/base`, `manifests/cluster-rbac` | command flags in `cmd/<component>/commands` |
| Generated files drift in CI | `Makefile` codegen targets | matching `hack/generate-*` or `hack/update-*` script |
| Notification does not fire/render | `notifications_catalog` | `notification_controller`, `util/notification` |
| Hydration/commit behavior fails | `controller/hydrator`, `util/hydrator` | `commitserver`, repo server |

## First-contribution workflow

1. Find an open, approved issue and keep the change inside its scope.
2. Identify the owning subsystem with the lookup table and read its local tests before editing.
3. Use `rg` to follow call sites and interfaces. Avoid trying to understand all of `util/` or all fixtures first.
4. Write the smallest behavior-focused change and add a regression test near the code. Add an e2e test only when the behavior genuinely crosses boundaries.
5. Update user/operator docs for a feature or visible behavior change.
6. If API types/protos/manifests/generated clients changed, run code generation and review every generated diff.
7. Before committing, follow `AGENTS.md`: `make build`, required `make codegen`, `make lint`, `make lint-ui`, `make test`, and `make cli`. These are heavy; targeted package/UI tests are useful during iteration but do not replace the required final checks.
8. Use a semantic PR title such as `fix: ...`, fully complete the PR template, reference the approved issue, and do not invent links or usernames.

The best first issue is usually localized to one package with existing tests. Changes spanning CRD types, controller reconciliation, generated clients/CRDs, manifests, API, UI, and docs are high-risk even when the visible feature looks small.
