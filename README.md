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
```

WhapCalendar variables managed:

```env
NEXT_PUBLIC_WHAP_URL=
NEXT_PUBLIC_WHAP_LOGIN_URL=
NEXT_PUBLIC_WHAP_PROFILE_URL=
WHAP_API_BASE_URL=
WHAPCALENDAR_WEBHOOK_SECRET=
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
3. Recreate containers after env changes. Do not rebuild WhapCalendar only for env changes unless code/build inputs changed.
4. Run `/home/greg/whapscripts/whap-env-sync runtime <dev|prod>` after containers are recreated.
5. Run `/home/greg/whapscripts/whap-env-sync smoke <dev|prod>` before marking deploy complete.

Never mix DEV and PROD secrets. Never copy variables between `/home/greg/apps/dev.whap.uy`, `/home/greg/apps/dev.whapcalendar.uy`, `/home/greg/apps/whap.uy`, and `/home/greg/apps/whapcalendar.uy` except through `whap-env-sync apply <env>` using the matching source file.
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
WHAPCALENDAR_PATH=/home/greg/apps/dev.whapcalendar.uy
WHAPCALENDAR_BRANCH=develop
WHAPCALENDAR_URL=https://dev.whap.uy:8444
WHAPCALENDAR_ENV_FILE=.env
WHAPCALENDAR_WEBHOOK_SECRET=replace-with-dev-secret
```

Expected PROD values:

```env
ENVIRONMENT=prod
WHAP_PATH=/home/greg/apps/whap.uy
WHAP_BRANCH=main
WHAP_URL=https://whap.uy
WHAP_APP_PORT=8001
WHAPCALENDAR_PATH=/home/greg/apps/whapcalendar.uy
WHAPCALENDAR_BRANCH=main
WHAPCALENDAR_URL=https://whap.uy:8443
WHAPCALENDAR_ENV_FILE=.env
WHAPCALENDAR_WEBHOOK_SECRET=replace-with-prod-secret
```
