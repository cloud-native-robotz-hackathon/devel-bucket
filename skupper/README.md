# Skupper

## Install skupper-cli
```bash
# Install skuppper-cli
# https://github.com/skupperproject/skupper/releases

curl -L -O https://github.com/skupperproject/skupper/releases/download/1.4.1/skupper-cli-1.4.1-linux-amd64.tgz
tar xzvf skupper-cli-1.4.1-linux-amd64.tgz  skupper
chmod +x skupper; sudo mv skupper /usr/local/bin/

```

## Deploy skuper in public cluster

```bash
$ oc new-project skupper-public
[...snipped...]
$ skupper init --enable-console --enable-flow-collector
clusterroles.rbac.authorization.k8s.io "skupper-service-controller" not found
Skupper is now installed in namespace 'skupper-public'.  Use 'skupper
status' to get more information.
$ skupper status
Skupper is enabled for namespace "skupper-public" in interior mode. It is not connected to any other sites. It has no exposed services.
The site console url is:  https://skupper-skupper-public.apps.demo.openshift.pub
The credentials for internal console-auth mode are held in secret: 'skupper-console-users'
$ oc get secret skupper-console-users -o jsonpath="{.data.admin}" | base64 -d; echo
G5xxxxxxx
$ skupper token create public.token
Token written to public.token
```

## 📛 Deploy skuper in private cluster (MicroShift)

=> Skupper is not availabel for ARM

<https://issues.redhat.com/browse/SKUPPER-667>

```bash

$ oc new-project skupper-private
[...snipped...]
$ skupper init
clusterroles.rbac.authorization.k8s.io "skupper-service-controller" not found
Skupper is now installed in namespace 'skupper-private'.  Use 'skupper status' to get more information.

$ oc get pods
NAME                                          READY   STATUS              RESTARTS   AGE
skupper-router-f48558885-29jd5                0/2     ContainerCreating   0          78s
skupper-service-controller-769577cc9c-9jzww   0/1     ContainerCreating   0          76s
$ oc get pods
NAME                                          READY   STATUS              RESTARTS   AGE
skupper-router-f48558885-29jd5                0/2     ContainerCreating   0          2m18s
skupper-service-controller-769577cc9c-9jzww   0/1     CrashLoopBackOff    1          2m16s
$ oc logs skupper-service-controller-769577cc9c-9jzww
exec container process `/app/service-controller`: Exec format error
$


```

🚨 Upstream skupper do not provide ARM images!


## Deploy skuper in private cluster (CRC/Local/SNO)

```bash
$ oc new-project skupper-private
[...snipped...]
$ skupper init
W0726 12:12:19.035967  748022 warnings.go:70] would violate PodSecurity "restricted:v1.24": allowPrivilegeEscalation != false (containers "router", "config-sync" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (containers "router", "config-sync" must set securityContext.capabilities.drop=["ALL"]), seccompProfile (pod or containers "router", "config-sync" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
W0726 12:12:21.010427  748022 warnings.go:70] would violate PodSecurity "restricted:v1.24": allowPrivilegeEscalation != false (container "service-controller" must set securityContext.allowPrivilegeEscalation=false), unrestricted capabilities (container "service-controller" must set securityContext.capabilities.drop=["ALL"]), seccompProfile (pod or container "service-controller" must set securityContext.seccompProfile.type to "RuntimeDefault" or "Localhost")
Skupper is now installed in namespace 'skupper-private'.  Use 'skupper status' to get more information.

$ skupper status
Skupper is enabled for namespace "skupper-private" in interior mode. It is not connected to any other sites. It has no exposed services.

$ skupper link create public.token
Site configured to link to https://claims-skupper-public.apps.demo.openshift.pub:443/360a6c99-2b9d-11ee-8ac2-701ab8659816 (name=link1)
Check the status of the link using 'skupper link status'.
$ skupper status
Skupper is enabled for namespace "skupper-private" in interior mode. It is connected to 1 other site. It has no exposed services.
```

