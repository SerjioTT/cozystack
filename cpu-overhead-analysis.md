# CPU Requests / Limits — Tenant Namespace Apps

> Анализ на основе репозитория [cozystack/cozystack](https://github.com/cozystack/cozystack)  
> Дата анализа: 2026-06-15  
> Версия репозитория: v1.4.3 (ветка main, коммит 25a6b56f)

## Правила расчёта

- `cpu.request = cpu.limit / cpuAllocationRatio`  
- Дефолт `cpuAllocationRatio = 10` (`packages/core/platform/values.yaml:321`)  
- Управляется через CR: `kind: Package, name: cozystack.cozystack-platform` → `spec.components.platform.values.resources.cpuAllocationRatio`  
- Значение пишется в `Secret cozystack-values` (`packages/core/platform/templates/apps.yaml:92`) и читается всеми чартами через `cozy-lib.loadCozyConfig`

## Пресеты ресурсов

Источник: `packages/library/cozy-lib/templates/_resourcepresets.tpl`

| Preset | CPU limit | Memory |
|--------|-----------|--------|
| t1.nano | 250m | 128Mi |
| t1.micro | 500m | 256Mi |
| t1.small | 1000m | 512Mi |
| t1.medium | 2000m | 1Gi |
| t1.large | 4000m | 2Gi |
| c1.nano | 250m | 256Mi |
| c1.micro | 500m | 512Mi |
| c1.small | 1000m | 1Gi |
| c1.medium | 2000m | 2Gi |
| c1.large | 4000m | 4Gi |
| s1.small | 1000m | 2Gi |
| s1.medium | 2000m | 4Gi |
| u1.small | 1000m | 4Gi |
| m1.small | 1000m | 8Gi |

Хелпер sanitize: `packages/library/cozy-lib/templates/_resources.tpl`  
- `cozy-lib.resources.sanitize` — строка 87  
- `cozy-lib.resources.defaultingSanitize` — строка 152

---

## Группа A — Управляемый overhead

> Контейнеры, CPU request которых вычисляется через `cozy-lib.resources.sanitize` / `cozy-lib.resources.defaultingSanitize`.  
> Реагируют на изменение `cpuAllocationRatio`.  
> **cpu.request / cpu.limit = 10% при дефолтном ratio=10**

### apps/clickhouse

Источник чарта: `packages/apps/clickhouse/`  
Дефолтные пресеты: `packages/apps/clickhouse/values.yaml`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| clickhouse | 1000m | 100m | t1.small | `templates/clickhouse.yaml:146` |
| clickhouse-keeper | 500m | 50m | t1.micro | `templates/chkeeper.yaml:61` |

### apps/foundationdb

Источник чарта: `packages/apps/foundationdb/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| foundationdb | 1000m | 100m | c1.small | `templates/cluster.yaml:70` |

### apps/harbor

Источник чарта: `packages/apps/harbor/`  
Дефолтные пресеты: `packages/apps/harbor/values.yaml`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| notary (хардкод preset) | 250m | 25m | t1.nano (hardcode) | `templates/harbor.yaml:120` |
| harbor-core | 1000m | 100m | t1.small | `templates/harbor.yaml:126` |
| harbor-registry (×2) | 1000m | 100m | t1.small | `templates/harbor.yaml:129,131` |
| harbor-jobservice | 250m | 25m | t1.nano | `templates/harbor.yaml:133` |
| harbor-trivy | 250m | 25m | t1.nano | `templates/harbor.yaml:137` |

### apps/http-cache

Источник чарта: `packages/apps/http-cache/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| haproxy | 250m | 25m | t1.nano | `templates/haproxy/deployment.yaml:36` |
| nginx | 250m | 25m | t1.nano | `templates/nginx/deployment.yaml:55` |

### apps/kafka

Источник чарта: `packages/apps/kafka/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| kafka broker | 1000m | 100m | c1.small | `templates/kafka.yaml:27` |
| zookeeper | 1000m | 100m | c1.small | `templates/kafka.yaml:83` |

### apps/kubernetes — Control Plane

Источник чарта: `packages/apps/kubernetes/`  
Дефолтные пресеты: `packages/apps/kubernetes/values.yaml` (поле `controlPlane.*`)

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| kube-apiserver | 2000m | 200m | c1.medium | `templates/cluster.yaml:180` |
| kube-controller-manager | 500m | 50m | t1.micro | `templates/cluster.yaml:182` |
| kube-scheduler | 500m | 50m | t1.micro | `templates/cluster.yaml:184` |
| konnectivity-server | 500m | 50m | t1.micro | `templates/cluster.yaml:198` |

> Примечание: control plane реплицируется (`controlPlane.replicas`, дефолт 2).

### apps/mariadb

Источник чарта: `packages/apps/mariadb/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| mariadb | 250m | 25m | t1.nano | `templates/mariadb.yaml:82` |

### apps/mongodb

Источник чарта: `packages/apps/mongodb/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| mongod (replica set) | 1000m | 100m | t1.small | `templates/mongodb.yaml:41` |
| mongos (при sharding=true) | 1000m | 100m | t1.small | `templates/mongodb.yaml:58` |
| configsvr (при sharding=true) | 1000m | 100m | t1.small | `templates/mongodb.yaml:72` |
| arbiter (при sharding=true) | 1000m | 100m | t1.small | `templates/mongodb.yaml:91` |

### apps/nats

Источник чарта: `packages/apps/nats/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| nats | 250m | 25m | t1.nano | `templates/nats.yaml:59` |

### apps/openbao

Источник чарта: `packages/apps/openbao/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| openbao | 1000m | 100m | t1.small | `templates/openbao.yaml:30` |

### apps/opensearch

Источник чарта: `packages/apps/opensearch/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| opensearch (data/master) | 2000m | 200m | c1.medium | `templates/opensearch.yaml:22` |
| opensearch (coordinator) | 2000m | 200m | c1.medium | `templates/opensearch.yaml:60` |
| opensearch-dashboards (опц.) | 1000m | 100m | c1.small | `templates/opensearch.yaml:43` |

### apps/postgres

Источник чарта: `packages/apps/postgres/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| postgresql (via CNPG) | 500m | 50m | t1.micro | `templates/db.yaml:192` |

### apps/qdrant

Источник чарта: `packages/apps/qdrant/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| qdrant | 1000m | 100m | t1.small | `templates/qdrant.yaml:28` |

### apps/rabbitmq

Источник чарта: `packages/apps/rabbitmq/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| rabbitmq | 250m | 25m | t1.nano | `templates/rabbitmq.yaml:15` |

### apps/redis

Источник чарта: `packages/apps/redis/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| redis | 250m | 25m | t1.nano | `templates/redisfailover.yaml:28` |
| sentinel | 250m | 25m | t1.nano | `templates/redisfailover.yaml:31` |

### apps/tcp-balancer

Источник чарта: `packages/apps/tcp-balancer/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| haproxy | 250m | 25m | t1.nano | `templates/deployment.yaml:37` |

### apps/vpn

Источник чарта: `packages/apps/vpn/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| openvpn | 250m | 25m | t1.nano | `templates/deployment.yaml:46` |

### extra/etcd

Источник чарта: `packages/extra/etcd/`  
Деплоится в tenant namespace через `apps/tenant/templates/etcd.yaml` при `etcd: true`.

| Контейнер | CPU limit | CPU request (ratio=10) | Источник значения | Источник шаблона |
|-----------|-----------|------------------------|-------------------|-----------------|
| etcd | 1000m | 100m | `values.yaml: resources.cpu: 1000m` | `templates/etcd-cluster.yaml:68` |

> Примечание: VPA bounds для etcd (`minAllowed: 250m`, `maxAllowed: 5000m`) захардкоданы в `templates/vpa.yaml:16-20`
> и также проходят через `sanitize`. При ratio=7 это вызывает баг (см. issue #2917).

### extra/external-dns

Источник чарта: `packages/extra/external-dns/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| external-dns | 250m | 25m | t1.nano | `templates/external-dns.yaml:166` |

### extra/ingress

Источник чарта: `packages/extra/ingress/`  
Деплоится в tenant namespace через `apps/tenant/templates/ingress.yaml` при `ingress: true`.

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| ingress-nginx-controller | 500m | 50m | t1.micro | `templates/nginx-ingress.yaml:50` |

### extra/seaweedfs

Источник чарта: `packages/extra/seaweedfs/`  
Деплоится в tenant namespace через `apps/tenant/templates/seaweedfs.yaml` при `seaweedfs: true`.

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| seaweedfs-db | 1000m | 100m | t1.small | `templates/seaweedfs-db.yaml:42` |
| seaweedfs-master | 1000m | 100m | t1.small | `templates/seaweedfs.yaml:143` |
| seaweedfs-volume (+ пулы) | 1000m | 100m | t1.small | `templates/seaweedfs.yaml:149,168,184,209` |
| seaweedfs-filer | 1000m | 100m | t1.small | `templates/seaweedfs.yaml:240` |
| seaweedfs-s3 | 1000m | 100m | t1.small | `templates/seaweedfs.yaml:267` |

### system/keycloak

Источник чарта: `packages/system/keycloak/`

| Контейнер | CPU limit | CPU request (ratio=10) | Preset | Источник шаблона |
|-----------|-----------|------------------------|--------|-----------------|
| keycloak | 1000m | 100m | t1.small (hardcode) | `templates/sts.yaml:115` |

---

## Группа B — Фиксированный неуправляемый overhead

> Контейнеры с **хардкоднутыми** ресурсами, **не проходящими** через `cozy-lib.resources.sanitize`.  
> **Не реагируют** на `cpuAllocationRatio`.  
> Присутствуют только при деплое `apps/kubernetes` (тенантный Kubernetes кластер).  
> Источник чарта: `packages/apps/kubernetes/`

### Cluster Autoscaler

Источник: `packages/apps/kubernetes/templates/cluster-autoscaler/deployment.yaml:42-46`

| Контейнер | CPU limit | CPU request | ratio req/limit | Управляем? |
|-----------|-----------|-------------|-----------------|------------|
| cluster-autoscaler | 512m | 125m | ~24.4% (≈1:4) | ❌ |

```yaml
# deployment.yaml:42-46
resources:
  limits:
    cpu: 512m
    memory: 512Mi
  requests:
    cpu: 125m
    memory: 128Mi
```

### Kubevirt Cloud Controller Manager (KCCM)

Источник: `packages/apps/kubernetes/templates/kccm/manager.yaml:41-48`

| Контейнер | CPU limit | CPU request | ratio req/limit | Управляем? |
|-----------|-----------|-------------|-----------------|------------|
| kubevirt-ccm | 512m | 125m | ~24.4% (≈1:4) | ❌ |

```yaml
# manager.yaml:41-48
resources:
  limits:
    cpu: 512m
    memory: 512Mi
  requests:
    cpu: 125m
    memory: 128Mi
```

### Kubevirt CSI Driver (KCSI)

Источник: `packages/apps/kubernetes/templates/csi/deploy.yaml`  
Все контейнеры в одном Deployment `<release>-kcsi-controller`.

| Контейнер | CPU limit | CPU request | ratio req/limit | Управляем? | Строки |
|-----------|-----------|-------------|-----------------|------------|--------|
| csi-driver | 512m | 125m | ~24.4% (≈1:4) | ❌ | `:69-76` |
| csi-provisioner | 512m | 125m | ~24.4% (≈1:4) | ❌ | `:83-90` |
| csi-attacher | 512m | 125m | ~24.4% (≈1:4) | ❌ | `:128-135` |
| csi-liveness-probe | 512m | 125m | ~24.4% (≈1:4) | ❌ | `:149-156` |
| csi-snapshotter | 512m | 125m | ~24.4% (≈1:4) | ❌ | `:172-179` |
| snapshot-controller | 512m | 125m | ~24.4% (≈1:4) | ❌ | `:196-203` |
| **csi-resizer** | **не задан** | **10m** | **∞ (no limit)** | ❌ | `:226-228` |

```yaml
# csi/deploy.yaml — типичный блок (csi-driver, provisioner, attacher, etc.)
resources:
  limits:
    cpu: 512m
  requests:
    cpu: 125m

# csi-resizer — исключение, limit не выставлен:
resources:
  requests:
    cpu: 10m
```

### Итог группы B — суммарный overhead на один kubernetes tenant

| Метрика | Значение |
|---------|----------|
| Контейнеров без sanitize | 9 |
| Суммарный CPU request (фиксированный) | 8 × 125m + 10m = **1010m ≈ 1 CPU** |
| Суммарный CPU limit | 8 × 512m = **4096m ≈ 4 CPU** (csi-resizer без limit) |
| Средний ratio req/limit (без csi-resizer) | 125 / 512 ≈ **24.4%** |

---

## Сводная статистика

| Группа | Контейнеров | % от всех | ratio cpu.req/cpu.limit |
|--------|-------------|-----------|-------------------------|
| A — управляемый (preset, ratio=10) | 35 | 81% | 10% (1:10) |
| B — фиксированный хардкод (k8s satellites) | 8 | 19% | ~24.4% (≈1:4) |
| B — без limit (csi-resizer) | 1 | — | N/A |
| **Среднее по 43 контейнерам с limit** | — | — | **≈ 12.7%** |

**Средний коэффициент перекоммита (limit/request) ≈ 7.9:1**

---

## Связанные issue

- [#2917](https://github.com/cozystack/cozystack/issues/2917) — VPA admission failure при нецелом milliCPU (`extra/etcd/templates/vpa.yaml`, `cozy-lib/templates/_resources.tpl:100`)
