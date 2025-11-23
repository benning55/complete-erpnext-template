## Install new extension or module
`
nano apps.json
`
Add the extension that you want
`
[
  {
    "url": "https://github.com/frappe/erpnext",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/hrms",
    "branch": "version-15"
  },
  {
    "url": "https://github.com/frappe/print_designer",
    "branch": "main"
  },
  {
    "url": "<https://github.com/frappe/example_of_extension>",
    "branch": "<Branch_of_the_repo>"
  },
]
`

## Build custom ready to use image
`
docker build --build-arg=FRAPPE_PATH=https://github.com/frappe/frappe --build-arg=FRAPPE_BRANCH=version-15 --build-arg=APPS_JSON_BASE64=$APPS_JSON_BASE64 --tag=domonit-erp-next:15 --file=images/custom/Containerfile .
`

## Custom create compose file and run it back
`
docker compose --env-file custom.env -f compose.yaml -f overrides/compose.mariadb.yaml -f overrides/compose.redis.yaml -f overrides/compose.noproxy.yaml config > compose.custom.yaml
docker compose -f compose.custom.yaml up -d
`

## Create new site
`
docker compose -p complete-erpnext-template exec backend bench new-site erp.domonit.de --mariadb-user-host-login-scope='172.%.%.%'
docker compose -p complete-erpnext-template exec backend bench --site erp.domonit.de  install-app erpnext
`

## Install custom app
`
docker compose -p complete-erpnext-template exec backend bench --site erp.domonit.de  install-app hrms
`

## Default username
`
Administrator
`

bench new-site erp.domonit.de --mariadb-user-host-login-scope='172.%.%.%'

## Check if scheduler work or not go to
`
RQ Job list
`

## Active job list
`
docker compose -p complete-erpnext-template exec backend bench --site erp.domonit.de enable-scheduler
`