# Rough notes

1. Log in to the web console as `admin` with the classic Red Hat demo password
1. Grab the token login command
1. `podman login -u thisisignored -p $(oc whoami -t) --tls-verify=false registry.apps.sno.demo.local`
