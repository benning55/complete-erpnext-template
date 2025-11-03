# Align Frappe And ERPNext Versions When Building The Custom Image

This guide walks through rebuilding the custom image so that the Frappe framework includes the `Currency Exchange Settings` DocType expected by ERPNext `version-15`.

## Prerequisites

- Access to the `complete-erpnext-template` project directory.
- Docker and Docker Compose installed on the target machine.
- Optional: ability to delete and recreate the ERPNext site inside the bench.

## 1. Verify Current Branch Pins

1. Open `apps.json` and confirm it still pins `frappe/erpnext`, `frappe/hrms`, and `frappe/print_designer` to the intended branches (`version-15`, `main`, etc.).
2. Decide whether to keep tracking the latest commits on those branches or to pin specific release tags. To stay current, keep the branch pins; to lock versions, change the entries to tags such as `v15.29.1`.

## 2. Rebuild The Image With A Fresh Frappe Checkout

1. Encode the updated `apps.json`:
   ```bash
   export APPS_JSON_BASE64="$(base64 -w 0 apps.json)"
   ```
2. Rebuild the custom image, forcing Docker to pull the latest base layers and git commits:
   ```bash
   docker build \
     --pull \
     --no-cache \
     --build-arg FRAPPE_PATH=https://github.com/frappe/frappe \
     --build-arg FRAPPE_BRANCH=version-15 \
     --build-arg APPS_JSON_BASE64="$APPS_JSON_BASE64" \
     --tag domonit-erp-next:15 \
     --file images/custom/Containerfile \
     .
   ```
   - Replace `FRAPPE_BRANCH` with a specific tag if you chose to pin releases in the previous step.

## 3. Regenerate The Compose Bundle (If Needed)

1. Recreate the combined compose file so it references the freshly built image:
   ```bash
   docker compose \
     --env-file custom.env \
     -f compose.yaml \
     -f overrides/compose.mariadb.yaml \
     -f overrides/compose.redis.yaml \
     -f overrides/compose.noproxy.yaml \
     config > compose.custom.yaml
   ```

## 4. Restart The Stack

1. Bring the services up using the regenerated compose file:
   ```bash
   docker compose -f compose.custom.yaml up -d
   ```
2. Wait for the containers—especially `backend`, `scheduler`, `queue-*`, and the database—to report healthy or running.

## 5. Recreate Or Update The Site

1. Enter the backend container:
   ```bash
   docker exec -it complete-erpnext-template-backend-1 bash
   ```
2. If the `frontend` site already exists and you want a clean install, drop it and recreate it (replace passwords as needed):
   ```bash
   bench drop-site frontend --force
   bench new-site frontend --mariadb-user-host-login-scope='172.%.%.%'
   ```
3. Install ERPNext (and any other apps) on the site:
   ```bash
   bench --site frontend install-app erpnext
   bench --site frontend install-app hrms
   bench --site frontend install-app print_designer
   ```
4. Exit the container when finished.

## 6. Smoke-Test The Installation

1. Check the bench logs to confirm the install completed without import errors:
   ```bash
   bench --site frontend doctor
   bench --site frontend list-apps
   ```
2. Open the site via the frontend service (default `http://SERVER_IP:8080`) and complete the ERPNext first-time setup wizard.

## Troubleshooting

- **Still seeing `Currency Exchange Settings` import errors:** Ensure you actually rebuilt the image with `--no-cache`; otherwise Docker may reuse the old Frappe checkout layer.
- **Git rate limits or connectivity issues during build:** Configure `git config --global --add url."https://<token>@github.com/".insteadOf "https://github.com/"` inside the build context or supply a mirror.
- **Database login failures after dropping the site:** Verify the MariaDB credentials in `custom.env` match what `bench new-site` expects.

Once Frappe and ERPNext come from aligned `version-15` commits, the DocType is present in core and the ERPNext install succeeds.
