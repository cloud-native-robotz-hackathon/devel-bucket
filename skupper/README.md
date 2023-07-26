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


## Deploy skupper in private cluster (CRC/Local/SNO)

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

## Service expose

 - hub controller is deployed on skupper-private (OpenShift Local / SNO )

### Service expose - PRIVATE ( OpenShift Local / SNO )

```bash

$ skupper expose deploymentconfig/hub-controller-live --port 8080
deploymentconfig hub-controller-live exposed as hub-controller-live

$ oc get svc hub-controller-live -o yaml | grep skupper
    internal.skupper.io/originalAssignedPort: 8080:1024
    internal.skupper.io/originalSelector: app=hub-controller-live,group=de.nexussix.openshift.hackathon,provider=fabric8
    internal.skupper.io/originalTargetPort: 8080:8080
  namespace: skupper-private
    application: skupper-router
    skupper.io/component: router
```

### Service - PUBLIC

```bash
$ oc get svc
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
hub-controller-live    ClusterIP   172.30.74.88     <none>        8080/TCP                     106s
[...snipped...]
```

🤩

Curl the robot
```bash
oc run curl --image=registry.access.redhat.com/ubi9-minimal --command -- curl http://hub-controller-live.skupper-public.svc.cluster.local:8080/api/robot/distance?user_key=terminator

oc logs -f curl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100     3  100     3    0     0      6      0 --:--:-- --:--:-- --:--:--     6
574
```
curl examples:
```bash
curl http://hub-controller-live-skupper-private.apps-crc.testing/robot/distance?user_key=terminator

curl -X POST -F 'user_key=terminator' http://hub-controller-live-skupper-private.apps-crc.testing/robot/forward/10

```