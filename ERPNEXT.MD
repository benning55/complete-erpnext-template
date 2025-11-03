## Build custom ready to use image
`
docker build --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe --build-arg=FRAPPE_BRANCH=version-15 --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 --tag=domonit-erp-next:15 --file=images/custom/Containerfile .
`

## Custom create compose file
`
docker compose --env-file custom.env -f compose.yaml -f overrides/compose.mariadb.yaml -f overrides/compose.redis.yaml -f overrides/compose.noproxy.yaml config > compose.custom.yaml
`

## Create new site
`
docker compose -p erpnext-template exec backend bench new-site frontend --mariadb-user-host-login-scope='172.%.%.%'
docker compose -p erpnext-template exec backend bench --site frontend install-app erpnext
`

## Install custom app
`
docker compose -p erpnext-template exec backend bench --site frontend install-app hrms
`

## Default username
`
Administrator
`