# Enrich logs data

Para "expandir" la información que llega de cada entrada de log, combinamos

- una politica para "enrich" data, con datos válidos
- un pipeline default para cada indice, que usará la politica para expandir la info
- un template para la creación automatica de los indices que setee esta configuracion

## Seteo de variables

```bash
ES_URL=https://elasticsearch-sample-es-http.elastic-system.svc:9200

```

## Crear Enrich policy

```bash
curl -XPUT "${ES_URL}/_enrich/policy/openshift_cluster" \
 -H 'Content-Type: application/json' \
 -d'
 {
   "match": {
     "indices": "openshift_servers",
     "match_field": "ip",
     "enrich_fields": ["cluster_id"]
    }
 }'

curl -XPOST "${ES_URL}/_enrich/policy/openshift_cluster/_execute"
```

### Crear Enrich data

Creamos un indice `openshift_servers` que contendrá los datos para hacer matching:

- hostname de los servidores
- ip fija
- cluster_id, con el string que queremos identificar al cluster

```bash
curl -XPOST "${ES_URL}/openshift_servers/_bulk" \
-H 'Content-Type: application/json' \
-d '{ "index" : {} }
{"hostname": "master-1", "ip": "10.1.0.1", "cluster_id": "cluster-1"}
{ "index" : {} }
{"hostname": "master-2", "ip": "10.1.0.2", "cluster_id": "cluster-1"}
{ "index" : {} }
{"hostname": "master-3", "ip": "10.1.0.3", "cluster_id": "cluster-1"}
{ "index" : {} }
{"hostname": "master-1", "ip": "10.2.0.1", "cluster_id": "cluster-2"}
{ "index" : {} }
{"hostname": "master-2", "ip": "10.2.0.2", "cluster_id": "cluster-2"}
{ "index" : {} }
{"hostname": "master-3", "ip": "10.2.0.3", "cluster_id": "cluster-2"}
{ "index" : {} }
{"hostname": "infra-1", "ip": "10.1.1.1", "cluster_id": "cluster-1"}
{ "index" : {} }
{"hostname": "infra-2", "ip": "10.1.1.2", "cluster_id": "cluster-1"}
{ "index" : {} }
{"hostname": "infra-3", "ip": "10.1.1.3", "cluster_id": "cluster-1"}
{ "index" : {} }
{"hostname": "infra-1", "ip": "10.2.1.1", "cluster_id": "cluster-2"}
{ "index" : {} }
{"hostname": "infra-2", "ip": "10.2.1.2", "cluster_id": "cluster-2"}
{ "index" : {} }
{"hostname": "infra-3", "ip": "10.2.1.3", "cluster_id": "cluster-2"}'
```

### Actualizar Enrich data

Si tenemos que agregar un nodo nuevo en uno de los clusters:

```bash
curl -XPOST "${ES_URL}/openshift_servers/_doc" \
-H 'Content-Type: application/json' \
-d '{"hostname": "infra-3", "ip": "10.2.1.3", "cluster_id": "cluster-2"}'
```

Si tenemos que actualizar un dato existente, 

- primero obtenemos el id
```bash
curl -XGET "${ES_URL}/openshift_servers/_search" \
-H 'Content-Type: application/json'
```

- luego ejecutamos un POST utilizando el ID obtenido

```bash
curl -XPOST "${ES_URL}/openshift_servers/_doc/<ID>" \
-H 'Content-Type: application/json' \
-d '{"hostname": "infra-3", "ip": "10.2.1.3", "cluster_id": "cluster-2"}'
```

## Crear Pipeline

```bash
curl -XPUT "${ES_URL}/_ingest/pipeline/cluster_id_pipeline" \
-H 'Content-Type: application/json' \
-d'{
  "match": {
    "indices": "openshift_servers",
    "match_field": "ip",
    "enrich_fields": ["cluster_id"]
  }
}'
```

## Crear template de los indices

https://www.elastic.co/guide/en/elasticsearch/reference/7.10/index-modules.html#dynamic-index-settings

```bash
curl -XPUT "${ES_URL}/_index_template/log_concentrator" \
-H 'Content-Type: application/json' \
-d '{
  "index_patterns": ["app-*", "infra-*", "audit-*"],
  "template": {
    "settings": {
      "number_of_shards": 3,
      "default_pipeline": "cluster_id_pipeline"
    },
    "mappings": {
      "properties": {
        "cluster_info" : {
          "properties" : {
            "cluster_id" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  },
  "priority": 20
}'
```

### Configurar Pipeline como default

si no usamos el template mencionado arriba, para una prueba rápida sobre un indice creado seteamos el pipeline default asi:

```bash
curl -XPUT "${ES_URL}/app-001/_settings" \
-H 'Content-Type: application/json' \
-d'{"default_pipeline": "cluster_id_pipeline"}'
```
