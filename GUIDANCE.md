# authentik on Wodby

What Wodby sets up for authentik on this service. authentik runs from the official `ghcr.io/goauthentik/server` image. The main service runs the server (`server`), which serves the web interface, the API and the sign-in flows on port 9000. The required `authentik-worker` derivative runs the same image as the worker (`worker`), which runs background tasks: blueprints, email, outposts, directory synchronization and cleanup. Configuration is passed as `AUTHENTIK_*` environment variables, and the worker receives the same variables as the server.

## Linked services

| Link | Variables |
| --- | --- |
| PostgreSQL (`postgres`, required) | `AUTHENTIK_POSTGRESQL__HOST`, `AUTHENTIK_POSTGRESQL__PORT`, `AUTHENTIK_POSTGRESQL__NAME`, `AUTHENTIK_POSTGRESQL__USER`, `AUTHENTIK_POSTGRESQL__PASSWORD` |
| Shared file storage (`storage`, required) | Mounts the shared `data` volume at `/data` on the server and the worker |
| SMTP server (`sendmail`, optional) | `AUTHENTIK_EMAIL__HOST`, `AUTHENTIK_EMAIL__PORT`, `AUTHENTIK_EMAIL__FROM`; TLS and SSL are off |

authentik keeps users, applications, providers, flows, sessions, its cache and its task queue in PostgreSQL. It does not use Redis. `AUTHENTIK_POSTGRESQL__SSLMODE` is `prefer`, so the connection is encrypted when the database server supports TLS.

Uploaded files, such as application icons, brand logos and flow backgrounds, and reports the worker writes are stored under `/data`. The server and the worker share that volume through the storage link.

## What the manifest sets

- `AUTHENTIK_SECRET_KEY` comes from the generated `secret_key` token. It signs cookies and sessions; changing it signs everyone out.
- `AUTHENTIK_WEB__BASE_URL` is the service's URL in the environment. authentik copies it into its system settings only when no base URL is stored yet, so after the domain changes, update the base URL in **System > Settings**.
- TLS ends at Wodby's router, which sets `X-Forwarded-Proto`. authentik trusts forwarded headers from private network addresses (its default).
- The update check, anonymous startup analytics and error reporting are off. New releases arrive as new versions of this service.

## First administrator

The Admin email and Admin password settings set `AUTHENTIK_BOOTSTRAP_EMAIL` and `AUTHENTIK_BOOTSTRAP_PASSWORD`. The first deployment creates the `akadmin` user in the `authentik Admins` group with them. Because that account has a password, authentik's public initial setup flow (`/if/flow/initial-setup/`) is closed. The administrator can sign in with `akadmin` or the email address. Changing the settings later does not change an existing account; change the password in authentik.

## Changing configuration

Add or change `AUTHENTIK_*` environment variables on the service and deploy it; the worker uses the same variables. Applications, providers, flows, policies and users are managed in the admin interface (`/if/admin/`) or the API (`/api/v3/`) and stored in the database.

## Upgrades

authentik migrates its database when a new version starts, and it refuses to skip a release. Move to the next version only, and back up the PostgreSQL database first; moving back to an older version is not supported.

## Check the result

From inside the environment, `/-/health/ready/` on port 9000 of the server is the readiness endpoint the service is probed with. The worker is checked with `ak healthcheck`.
