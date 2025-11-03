# Comandi OpenSearch / Wazuh — Documentazione Tecnica

---

## Cluster — Stato e Allocazione

> Stato di salute del cluster e di ogni indice (`green`, `yellow`, `red`).
```http
GET /_cluster/health?level=indices
```
> Stato generale del cluster.
```http
GET /_cluster/health
```

> Spiega perché uno shard non è stato allocato (utile per indici in stato `red`).
```http
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


```http
GET /_cat/segments/wazuh-archives-4.x-*?v&h=index,shard,segment,count,size
```
> Segmenti per shard: utile per capire la frammentazione.

---

## 📦 Indici — Stato, Dimensione, Documenti

```bash
GET _cat/indices?v
```
> Elenco completo degli indici.

```bash
GET _cat/indices?v&h=index,health,status,store.size
```
> Stato e dimensione di ogni indice.

```bash
GET _cat/indices?v&h=index,docs.count,store.size,health
```
> Numero documenti, dimensione e stato di salute.

```bash
GET /wazuh-archives-4.x-*/_settings?include_defaults=true
```
> Impostazioni degli indici `wazuh-archives`.

```bash
GET /_stats/store
```
> Statistiche di spazio occupato per tutti gli indici.

---

##  Archivi Wazuh — Pulizia e Gestione

```bash
GET /_cat/indices/wazuh-archives-4.x-*?v&h=index,docs.count,store.size
```
> Verifica documenti e spazio occupato dagli archivi.
> 
> Mostra il documento più vecchio:
```bash
GET /wazuh-archives-4.x-*/_search
```


```json
{
  "size": 1,
  "sort": [{ "@timestamp": "asc" }],
  "_source": ["@timestamp"]
}
```

```bash
POST /wazuh-archives-4.x-*/_delete_by_query
```
> Cancella documenti più vecchi di 6 mesi:

```json
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

```bash
POST /wazuh-archives-4.x-*/_delete_by_query?conflicts=proceed&wait_for_completion=false
```
> Cancellazione parallela con slice:

```json
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

```bash
POST /wazuh-archives-4.x-*/_forcemerge?only_expunge_deletes=true
```
> Compatta segmenti e libera spazio disco.

```bash
GET /_tasks?actions=indices:admin/forcemerge&detailed=true
```
> Stato dei task `forcemerge`.

```bash
GET /_tasks?actions=*delete
```
> Stato dei task `delete_by_query`.

```bash
GET /_tasks/<task_id>
```
> Stato di un task specifico.

```bash
POST /_tasks/<task_id>/_cancel
```
> Cancella un task attivo.

---

##  Opendistro / ISM — Configurazione e Ripristino

```bash
GET _cat/indices/.opendistro-ism-config?v
GET _cat/shards/.opendistro-ism-config?v
```
> Stato dell’indice ISM e dei suoi shard.

```bash
DELETE /.opendistro-ism-config
```
> Cancella l’indice ISM (utile se shard primario è `unassigned`).

```bash
PUT /.opendistro-ism-config
```
> Ricrea l’indice ISM:

```json
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
```

```bash
PUT /.opendistro-job-scheduler-lock
```
> Ricrea l’indice di lock per job scheduler.

```bash 
PUT /.opendistro-job-scheduler-lock/_settings
```
> Imposta repliche a 0.

```bash
DELETE /.opendistro-job-scheduler-lock
```
> Cancella l’indice di lock (se in stato `red`).

```bash
PUT /.opendistro-ism-managed-index-history-*/_settings
```
> Imposta repliche a 0 per cronologia ISM.

---

## Opendistro Alerting — Indici e Repliche

```bash
GET _cat/indices/.opendistro-alerting-alerts?v&h=index,health,status
GET _cat/shards/.opendistro-alerting-alerts?v
```
> Stato dell’indice e shard di alerting.

```bash
PUT /.opendistro-alerting-alerts/_settings
```
> Imposta repliche a 0 (può fallire se Wazuh blocca la modifica).

---

## Sicurezza / Ruoli — Permessi e Mapping

```bash
GET _plugins/_security/api/rolesmapping/admin
GET _plugins/_security/api/roles/admin
GET _plugins/_security/api/tenants
```
> Visualizza ruoli, mapping e tenant.

```bash
PUT _plugins/_security/api/roles/admin
```
> Imposta permessi completi per `admin`:

```json
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

```bash
PUT _plugins/_security/api/rolesmapping/admin
```
> Associa l’utente `admin` al ruolo `admin`:

```json
{
  "users": ["admin"],
  "backend_roles": ["admin"],
  "hosts": []
}
```

---

📘 *File di riferimento per manutenzione, diagnostica e gestione OpenSearch/Wazuh.*
