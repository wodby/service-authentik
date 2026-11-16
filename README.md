# authentik service for Kubernetes on Wodby

Run authentik as a reusable Kubernetes application service with Wodby.

This repository defines the Wodby service manifests and operational
configuration for authentik.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [authentik application stack](https://github.com/wodby/stack-authentik)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `authentik` |
| Type | Application service |
| Versions | `2026.8` by default; also available: `2026.5` |
| Workloads | `main` (Deployment, primary) |
| Containers | `authentik` using `ghcr.io/goauthentik/server` |
| Endpoints | `authentik`: HTTP 9000 (main) |
| Service links | PostgreSQL (`postgres`), required; Shared file storage (`storage`), required; SMTP server (`sendmail`), optional |
| Application build | Not buildable from application source |
| Helm | chart `oci://registry-1.docker.io/wodby/stateless`; version `0.4.0` |
| Configuration and operations | 2 settings, 1 volumes |

## Use this service

Use this service through [authentik application stack](https://github.com/wodby/stack-authentik), or reference `authentik` from a
custom Wodby stack.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Server and worker

The main service runs the authentik server, which serves the web interface,
the API and the sign-in flows. The required `authentik-worker` derivative runs
the same image as the authentik worker, which runs background tasks such as
blueprints, email and directory synchronization. Both share the PostgreSQL
database and the shared file storage that holds uploaded icons, logos and
reports.

## First administrator

The first deployment creates the `akadmin` administrator from the service's
required Admin email and Admin password settings, so authentik's initial setup
page is never open to the public. Changing the settings later does not change
an existing account.

## Upgrades

authentik migrates its database when a newer version starts and refuses to
skip a release. Move to the next version only, and back up the PostgreSQL
database first; moving back to an older version is not supported.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
