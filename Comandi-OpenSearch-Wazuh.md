# Comandi OpenSearch / Wazuh — Documentazione Tecnica

---

## Cluster — Stato e Allocazione

> Stato di salute del cluster e di ogni indice (`green`, `yellow`, `red`).
```bash
GET /_cluster/health?level=indices
```
> Stato generale del cluster.
```bash
GET /_cluster/health
```

> Spiega perché uno shard non è stato allocato (utile per indici in stato `red`).
```bash
GET /_cluster/allocation/explain?pretty
```

> Forza l’allocazione di uno shard primario non assegnato:
```bash
POST /_cluster/reroute
{
  "commands": [
    {
      "allocate_stale_primary": {
        "index": "<indice>",
        "shard": 0,
        "node": "node-wazuh",
        "accept_data_loss": true
      }
    }
  ]
}
```

---

##  Shards — Stato, Allocazione, Segmenti

> Stato shard e motivi di mancata assegnazione.
```bash
GET _cat/shards?v&h=index,shard,prirep,state,unassigned.reason
```

> Stato shard e nodo assegnato.
```bash
GET _cat/shards?v&h=index,shard,prirep,state,node
```

> Stato shard per gli indici `wazuh-archives`.
```bash
GET /_cat/shards/wazuh-archives-4.x-*?v
```

> Segmenti per shard: utile per capire la frammentazione.
```bash
GET /_cat/segments/wazuh-archives-4.x-*?v&h=index,shard,segment,count,size
```


---

## Indici — Stato, Dimensione, Documenti

> Elenco completo degli indici.
```bash
GET _cat/indices?v
```

> Stato e dimensione di ogni indice.
```bash
GET _cat/indices?v&h=index,health,status,store.size
```
> Numero documenti, dimensione e stato di salute.

```bash
GET _cat/indices?v&h=index,docs.count,store.size,health
```
> Impostazioni degli indici `wazuh-archives`.

```bash
GET /wazuh-archives-4.x-*/_settings?include_defaults=true
```
> Statistiche di spazio occupato per tutti gli indici.

```bash
GET /_stats/store
```

---

##  Archivi Wazuh — Pulizia e Gestione

> Verifica documenti e spazio occupato dagli archivi.
```bash
GET /_cat/indices/wazuh-archives-4.x-*?v&h=index,docs.count,store.size
```

> Mostra il documento più vecchio:
```bash
GET /wazuh-archives-4.x-*/_search
{
  "size": 1,
  "sort": [{ "@timestamp": "asc" }],
  "_source": ["@timestamp"]
}
```
> Cancella documenti più vecchi di 6 mesi:

```bash
POST /wazuh-archives-4.x-*/_delete_by_query
{
  "query": {
    "range": {
      "@timestamp": {
        "lt": "now-6M/M"
      }
    }
  }
}
```

> Cancellazione parallela con slice:

```bash
POST /wazuh-archives-4.x-*/_delete_by_query?conflicts=proceed&wait_for_completion=false
{
  "slice": { "id": 0, "max": 10 },
  "query": {
    "range": {
      "@timestamp": {
        "lt": "now-6M/M"
      }
    }
  }
}
```
> Compatta segmenti e libera spazio disco.
```bash
POST /wazuh-archives-4.x-*/_forcemerge?only_expunge_deletes=true
```
> Stato dei task `forcemerge`.
```bash
GET /_tasks?actions=indices:admin/forcemerge&detailed=true
```
> Stato dei task `delete_by_query`.
```bash
GET /_tasks?actions=*delete
```
> Stato di un task specifico.
```bash
GET /_tasks/<task_id>
```
> Cancella un task attivo.
```bash
POST /_tasks/<task_id>/_cancel
```

---

##  Opendistro / ISM — Configurazione e Ripristino

> Stato dell’indice ISM e dei suoi shard.
```bash
GET _cat/indices/.opendistro-ism-config?v
GET _cat/shards/.opendistro-ism-config?v
```
> Cancella l’indice ISM (utile se shard primario è `unassigned`).
```bash
DELETE /.opendistro-ism-config
```

> Ricrea l’indice ISM:
```bash
PUT /.opendistro-ism-config
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```
> Ricrea l’indice di lock per job scheduler.
```bash
PUT /.opendistro-job-scheduler-lock
```
> Imposta repliche a 0.
```bash 
PUT /.opendistro-job-scheduler-lock/_settings
```
> Cancella l’indice di lock (se in stato `red`).
```bash
DELETE /.opendistro-job-scheduler-lock
```
> Imposta repliche a 0 per cronologia ISM.
```bash
PUT /.opendistro-ism-managed-index-history-*/_settings
```

---

## Opendistro Alerting — Indici e Repliche

> Stato dell’indice e shard di alerting.
```bash
GET _cat/indices/.opendistro-alerting-alerts?v&h=index,health,status
GET _cat/shards/.opendistro-alerting-alerts?v
```
> Imposta repliche a 0 (può fallire se Wazuh blocca la modifica).
```bash
PUT /.opendistro-alerting-alerts/_settings
```

---

## Sicurezza / Ruoli — Permessi e Mapping

```bash
GET _plugins/_security/api/rolesmapping/admin
GET _plugins/_security/api/roles/admin
GET _plugins/_security/api/tenants
```
> Visualizza ruoli, mapping e tenant.

> Imposta permessi completi per `admin`:
```bash
PUT _plugins/_security/api/roles/admin
{
  "cluster_permissions": ["cluster_all"],
  "index_permissions": [
    {
      "index_patterns": ["*"],
      "allowed_actions": ["indices_all"]
    }
  ],
  "tenant_permissions": [
    {
      "tenant_patterns": ["global_tenant", "admin_tenant"],
      "allowed_actions": ["kibana_all_read", "kibana_all_write"]
    }
  ]
}
```
> Associa l’utente `admin` al ruolo `admin`:
```bash
PUT _plugins/_security/api/rolesmapping/admin
{
  "users": ["admin"],
  "backend_roles": ["admin"],
  "hosts": []
}
```

---

📘 *File di riferimento per manutenzione, diagnostica e gestione OpenSearch/Wazuh.*
