# Elasticsearch images for OpenShift

Build image a partir de un Dockerfile

```bash
alias oc=oc-3.11.59
oc login https://api-ocp.migraciones.cloud 
# crear la imagen ES
oc new-build https://github.com/hugomelendez/docker-rhel-elasticsearch --strategy=docker --name=elasticsearch-rhel7-sma
# luego de esto queda creado un Build Config que se puede usar para buildear continuamenete

# crear la app ES en base a esta nueva imagen
oc new-app elasticsearch-rhel7-sma --name=elasticsearch
```

## Levantar aplicacíon ElasticSearch en un proyecto

Con la imagen elasticsearch-rhel7-sma creada...

```bash
oc project es-test
```

### Creacion de la SA *elastic* para ejecutar los pods

Se requiere una SA con policy especial para montar el volumen de los datos por nodo

```bash
oc create -f elasticsearch-deployment/elastic-sa.yaml
oc adm policy add-scc-to-user hostmount-anyuid -z elasticsearch
```

## TEMPORAL: Creación del deployment config para Elastic Search por Nodo en base a un template, luego se reemplazará por el StatefulSet

```bash
oc create -f elasticsearch-node-template.yaml

oc new-app --template=elasticsearch-node-template \
  --param PARAM_ES_NODE_NAME=elasticsearch-master-01 \
  --param PARAM_ES_NODE_AFFINITY=elasticsearch-master \
  --param PARAM_ES_IS_NODE_MASTER=true \
  --param PARAM_ES_IS_NODE_DATA=false \
  --param PARAM_ES_CLUSTER_NAME=elasticsearch-sma \
  --param PARAM_ES_NODE_ROLE=master

oc new-app --template=elasticsearch-node-template \
  --param PARAM_ES_NODE_NAME=elasticsearch-data-01 \
  --param PARAM_ES_NODE_AFFINITY=elasticsearch-data-01 \
  --param PARAM_ES_IS_NODE_MASTER=false \
  --param PARAM_ES_IS_NODE_DATA=true \
  --param PARAM_ES_CLUSTER_NAME=elasticsearch-sma \
  --param PARAM_ES_NODE_ROLE=client

oc new-app --template=elasticsearch-node-template \
  --param PARAM_ES_NODE_NAME=elasticsearch-data-02 \
  --param PARAM_ES_NODE_AFFINITY=elasticsearch-data-02 \
  --param PARAM_ES_IS_NODE_MASTER=false \
  --param PARAM_ES_IS_NODE_DATA=true \
  --param PARAM_ES_CLUSTER_NAME=elasticsearch-sma \
  --param PARAM_ES_NODE_ROLE=client
```

```bash
oc delete template elasticsearch-node-template
```

## Creacion de los servicios API (client) y Discovery (master) para exponer los puertos 9200 (api externa) y 9300 (api discovery para los demas nodos)

```bash
oc create -f elasticsearch-cluster-service.yaml
oc create -f elasticsearch-api-service.yaml
```