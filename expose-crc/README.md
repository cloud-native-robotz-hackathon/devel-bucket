# Expose CRC - AS ROOT!!!

```bash

export PUBLIC_IP=10.89.114.31
export CRC_IP=192.168.130.11

export HAPROXY_CFG="
global
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats


$STATS_CFG

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------

defaults
    mode                    http
    log                     global
    #option                  httplog
    option                  dontlognull
    option                  http-server-close
    #option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

listen ingress-http
    bind ${PUBLIC_IP}:80
    mode tcp
    server crc ${CRC_IP}:80 check inter 1s

listen ingress-https
    bind ${PUBLIC_IP}:443
    mode tcp
    server crc ${CRC_IP}:443 check inter 1s

listen api
    bind ${PUBLIC_IP}:6443
    mode tcp
    server crc ${CRC_IP}:6443 check inter 1s
"

podman run --env-host --net host -ti --rm quay.io/redhat-emea-ssa-team/openshift-4-loadbalancer:latest

```