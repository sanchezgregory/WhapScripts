# WhapScripts

Utilities for keeping Whap and WhapCalendar integration environment variables synchronized.

## Local Usage

The local source of truth is:

```text
/Users/gregorysanchez/projects/WhapScripts/.whap-integration.local.env
```

Run checks:

```bash
/Users/gregorysanchez/projects/WhapScripts/whap-env-sync check local
```

Apply integration variables to both project `.env` files:

```bash
/Users/gregorysanchez/projects/WhapScripts/whap-env-sync apply local
```

Rotate all four integration secrets and synchronize both projects without
printing their values:

```bash
/Users/gregorysanchez/projects/WhapScripts/whap-env-sync rotate local
```

Validate running containers when available:

```bash
/Users/gregorysanchez/projects/WhapScripts/whap-env-sync runtime local
```

Smoke test configured URLs:

```bash
/Users/gregorysanchez/projects/WhapScripts/whap-env-sync smoke local
```

## Scope

The script only manages Whap/WhapCalendar integration variables.

Whap variables managed:

```env
WHAPCALENDAR_URL=
WHAPCALENDAR_PUBLIC_URL=
WHAPCALENDAR_INTERNAL_URL=
WHAPCALENDAR_WEBHOOK_SECRET=
WHAPCALENDAR_API_SECRET=
WHAPCALENDAR_SSO_SECRET=
WHAPCALENDAR_CONTEXT_SECRET=
```

WhapCalendar variables managed:

```env
NEXT_PUBLIC_WEBAPP_URL=
NEXT_PUBLIC_WEBSITE_URL=
NEXTAUTH_URL=
WEB_APP_URL=
NEXT_PUBLIC_EMBED_LIB_URL=
ALLOWED_HOSTNAMES=
NEXT_PUBLIC_WHAP_URL=
NEXT_PUBLIC_WHAP_LOGIN_URL=
NEXT_PUBLIC_WHAP_PROFILE_URL=
WHAP_API_BASE_URL=
WHAPCALENDAR_WEBHOOK_SECRET=
WHAPCALENDAR_API_SECRET=
WHAPCALENDAR_SSO_SECRET=
WHAPCALENDAR_CONTEXT_SECRET=
```

It does not manage database credentials, app keys, OAuth secrets, Stripe, MercadoPago, NextAuth, JWT, or encryption keys.

## VPS Extension

On the VPS, this repository should live at:

```text
/home/greg/whapscripts
```

Hermes and any deploy agent must treat this script as the source-of-truth validator for Whap/WhapCalendar integration env values.

Before any DEV or PROD integration deploy, run:

```bash
/home/greg/whapscripts/whap-env-sync check dev
```

or:

```bash
/home/greg/whapscripts/whap-env-sync check prod
```

If the check fails because integration variables are out of sync, run the matching apply command before building or recreating containers:

```bash
/home/greg/whapscripts/whap-env-sync apply dev
/home/greg/whapscripts/whap-env-sync apply prod
```

After env changes and container recreation, validate runtime and smoke checks:

```bash
/home/greg/whapscripts/whap-env-sync runtime dev
/home/greg/whapscripts/whap-env-sync smoke dev
```

or:

```bash
/home/greg/whapscripts/whap-env-sync runtime prod
/home/greg/whapscripts/whap-env-sync smoke prod
```

Add these source files when using the same script on the VPS:

```text
/home/greg/whapscripts/.whap-integration.dev.env
/home/greg/whapscripts/.whap-integration.prod.env
```

These files must not be committed. They are ignored by `.gitignore` because they contain shared secrets.

`WHAPCALENDAR_WEBHOOK_SECRET`, `WHAPCALENDAR_API_SECRET`, `WHAPCALENDAR_SSO_SECRET`, and `WHAPCALENDAR_CONTEXT_SECRET` are shared integration secrets used by both Whap and WhapCalendar. They must each contain at least 32 characters, be distinct, and match in the Whap `.env`, the active WhapCalendar env file, and both runtimes. These values must only come from the matching source-of-truth file and must never be generated independently by a deploy script. Use `whap-env-sync rotate <environment>` for coordinated rotation and synchronization.

`WHAP_API_BASE_URL` is the server-side URL WhapCalendar uses to call Whap. If it is omitted from the source file, `whap-env-sync` falls back to `http://whap/api` for local and VPS. Define `WHAP_API_BASE_URL` in the source file when an environment needs a different route.

Local Docker development expects Whap and WhapCalendar to share the external Docker network `whap-shared-testing`. Create it once before starting either app:

```bash
docker network create whap-shared-testing
```

Whap already attaches `laravel.test` to this network with alias `whap`. WhapCalendar `docker-compose.dev.yml` also attaches dev containers to the same network, so local WC server-side calls should use `WHAP_API_BASE_URL=http://whap/api`.

For local Whap server-side calls back to WhapCalendar, use `WHAPCALENDAR_INTERNAL_URL=http://whapcalendar:3000`. Browser-visible WhapCalendar URLs should remain `http://localhost:3000`.

## Docker Cleanup

Use the shared Docker cleanup helper when WhapCalendar or Whap Docker builds fail with `no space left on device`, or before a large rebuild on a cramped VPS:

```bash
/home/greg/whapscripts/whap-docker-clean status
/home/greg/whapscripts/whap-docker-clean clean
```

For non-interactive deploy scripts:

```bash
/home/greg/whapscripts/whap-docker-clean clean --yes
```

The cleanup removes Docker build cache, stopped containers, unused images, and unused BuildKit cache. It does not remove Docker volumes, so database volumes are left intact. The next image build may be slower because Docker must regenerate cache layers.

WhapCalendar deploys can invoke this pre-deploy cleanup with:

```bash
./scripts/wc-up.sh --target vps --mode testing --strategy rebuild --clean-docker
```

## Hermes Deploy Agent Rule

Add this rule to the Hermes deploy prompt:

```md
## WhapScripts Integration Env Rule

The integration env source-of-truth script lives on the VPS at:

`/home/greg/whapscripts/whap-env-sync`

Before any Whap/WhapCalendar integration deploy, identify the target environment: `dev` or `prod`.

DEV pair:
- Whap: `/home/greg/apps/dev.whap.uy`
- WhapCalendar: `/home/greg/apps/dev.whapcalendar.uy`
- WhapCalendar URL: `https://dev.whap.uy:8444`
- Env sync source: `/home/greg/whapscripts/.whap-integration.dev.env`

PROD pair:
- Whap: `/home/greg/apps/whap.uy`
- WhapCalendar: `/home/greg/apps/whapcalendar.uy`
- WhapCalendar URL: `https://whap.uy:8443`
- Env sync source: `/home/greg/whapscripts/.whap-integration.prod.env`

Mandatory flow:
1. Run `/home/greg/whapscripts/whap-env-sync check <dev|prod>` before build/deploy.
2. If check fails because integration env values differ, run `/home/greg/whapscripts/whap-env-sync apply <dev|prod>`.
3. Confirm `WHAPCALENDAR_WEBHOOK_SECRET`, `WHAPCALENDAR_API_SECRET`, `WHAPCALENDAR_SSO_SECRET`, and `WHAPCALENDAR_CONTEXT_SECRET` have the same hashes in Whap and WhapCalendar. If any differs, stop; do not deploy.
4. Recreate containers after env changes. Do not rebuild WhapCalendar only for env changes unless code/build inputs changed.
5. After any WhapCalendar rebuild or container recreation, verify the paired Whap containers are still running. If `laravel.test` is stopped, start Whap with `docker compose up -d` in the matching Whap project. The WC helper `scripts/wc-up.sh` performs this check automatically for VPS `up` actions.
6. Run `/home/greg/whapscripts/whap-env-sync runtime <dev|prod>` after containers are recreated.
7. Run `/home/greg/whapscripts/whap-env-sync smoke <dev|prod>` before marking deploy complete.

Never mix DEV and PROD secrets. Never copy variables between `/home/greg/apps/dev.whap.uy`, `/home/greg/apps/dev.whapcalendar.uy`, `/home/greg/apps/whap.uy`, and `/home/greg/apps/whapcalendar.uy` except through `whap-env-sync apply <env>` using the matching source file. Never let `wc-up.sh` or any deploy step generate random WhapCalendar integration secrets on the VPS.
```

Alternative local-only layout for development machines:

```text
.whap-integration.dev.env
.whap-integration.prod.env
```

Expected DEV values:

```env
ENVIRONMENT=dev
WHAP_PATH=/home/greg/apps/dev.whap.uy
WHAP_BRANCH=develop
WHAP_URL=https://dev.whap.uy
WHAP_APP_PORT=8002
WHAP_API_BASE_URL=http://whap/api
WHAPCALENDAR_PATH=/home/greg/apps/dev.whapcalendar.uy
WHAPCALENDAR_BRANCH=develop
WHAPCALENDAR_URL=https://dev.whap.uy:8444
WHAPCALENDAR_ENV_FILE=.env.wc.vps.testing
WHAPCALENDAR_WEBHOOK_SECRET=replace-with-dev-secret
WHAPCALENDAR_API_SECRET=replace-with-dev-secret
WHAPCALENDAR_SSO_SECRET=replace-with-dev-secret
WHAPCALENDAR_CONTEXT_SECRET=replace-with-dev-secret
```

Expected PROD values:

```env
ENVIRONMENT=prod
WHAP_PATH=/home/greg/apps/whap.uy
WHAP_BRANCH=main
WHAP_URL=https://whap.uy
WHAP_APP_PORT=8001
WHAP_API_BASE_URL=http://whap/api
WHAPCALENDAR_PATH=/home/greg/apps/whapcalendar.uy
WHAPCALENDAR_BRANCH=main
WHAPCALENDAR_URL=https://whap.uy:8443
WHAPCALENDAR_ENV_FILE=.env.wc.vps.production
WHAPCALENDAR_WEBHOOK_SECRET=replace-with-prod-secret
WHAPCALENDAR_API_SECRET=replace-with-prod-secret
WHAPCALENDAR_SSO_SECRET=replace-with-prod-secret
WHAPCALENDAR_CONTEXT_SECRET=replace-with-prod-secret
```
