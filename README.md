# Elasticsearch images for OpenShift


Build image a partir de un Dockerfile

```bash
alias oc=oc-3.5.5.8
# crear la imagen ES
oc new-build https://github.com/hmelendez/docker-rhel-elasticsearch --strategy=docker --name=elasticsearch-rhel7-sma
# luego de esto queda creado un Build Config que se puede usar para buildear continuamenete

# crear la app ES en base a esta nueva imagen
oc new-app elasticsearch-rhel7-sma --name=elasticsearch

```







[![Build Status](https://travis-ci.org/RHsyseng/docker-rhel-elasticsearch.svg?branch=master)](https://travis-ci.org/RHsyseng/docker-rhel-elasticsearch)

This image includes a default jvm configuration to get the Heap configuration from the cgroups.

 * [RHEL 7.3 image](./Dockerfile)
 * [CentOS 7 image](./Dockerfile.centos7)
 * Installation of Elasticsearch 6.2.2 is done through RPMs
 * The plugin is included directly as binary because it is not yet available on Maven Central

An example deployment file is also included:
```
$ oc create -f es-cluster-deployment.yml
statefulset "elasticsearch" created
service "elasticsearch" created
service "elasticsearch-cluster" created
imagestream "elasticsearch" created
```

## Delete all resources
```bash
$ oc delete all -l app=elasticsearch
```
