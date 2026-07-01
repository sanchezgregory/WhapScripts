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

Add these files when using the same script on the VPS:

```text
/home/greg/.whap-integration.dev.env
/home/greg/.whap-integration.prod.env
```

Or keep them beside this script as:

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
