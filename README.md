## Вопрос 1

- текст Dockerfile манифеста
```
FROM centos:7
RUN useradd elastic
RUN mkdir /elasticsearch && mkdir /var/lib/elasticsearch
ADD elasticsearch-8.2.3-linux-x86_64.tar.gz /elasticsearch
COPY elasticsearch.yml /elasticsearch/elasticsearch-8.2.3/config/
RUN chown -R elastic:elastic /elasticsearch && chown -R elastic:elastic /var/lib/elasticsearch
USER elastic
EXPOSE 9200/tcp
ENTRYPOINT /elasticsearch/elasticsearch-8.2.3/bin/elasticsearch
```
текст elasticsearch.yml
```
node.name: netology-test
path.data: /var/lib/elasticsearch
path.repo: /elasticsearch/elasticsearch-8.2.3
network.host: 0.0.0.0
http.port: 9200
```
- ссылка на образ в репозитории https://hub.docker.com/repository/docker/dmitrii1980/elasticsearch

после запуска контейнера нужно сбросить пароль суперпользователя

- ответ elasticsearch на запрос пути / в json виде
```
{
  "name" : "netology-test",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "M1JfusVVSumJT5TlXtaLEA",
  "version" : {
    "number" : "8.2.3",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "9905bfb62a3f0b044948376b4f607f70a8a151b4",
    "build_date" : "2022-06-08T22:21:36.455508792Z",
    "build_snapshot" : false,
    "lucene_version" : "9.1.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```
## Вопрос 2
- Получите список индексов и их статусов, используя API и приведите в ответе на задание.
```
green  open ind-1 gXFl4FAcQHCId5fljqWptw 1 0 0 0 225b 225b
yellow open ind-3 Z7vl7MmsSSigYjB_GVZ3Cg 4 2 0 0 900b 900b
yellow open ind-2 _FNSgpWiQVq4YsyRtOVB1w 2 1 0 0 450b 450b
```
- Получите состояние кластера elasticsearch, используя API.
```
{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 9,
  "active_shards" : 9,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 47.368421052631575
}
```
- Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

В кластере сейчас один хост, и некуда делать реплики. Поэтому первый индекс ind-1, настроенный без реплик, находится в зелёном состоянии, а остальные индексы настроены с репликами, но делать их некуда. А так как активные шарды доступны, то индексы в жёлтом состоянии.

## Вопрос 3

- Приведите в ответе запрос API и результат вызова API для создания репозитория.
```
# curl -k -u elastic -X PUT "https://localhost:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d'
{
  "type": "fs",
  "settings": {
    "location": "/elasticsearch/elasticsearch-8.2.3/snapshot"
  }
}
'
{
  "acknowledged" : true
}
```
- Создайте индекс test с 0 реплик и 1 шардом и приведите в ответе список индексов.
```
green open test jysFpaj7Tqqd0oqi6Trmlg 1 0 0 0 225b 225b
```

- Приведите в ответе список файлов в директории со snapshotами
```
$ ls -l /elasticsearch/elasticsearch-8.2.3/snapshot/
total 36
-rw-r--r-- 1 elastic elastic  1092 Jun 22 06:27 index-0
-rw-r--r-- 1 elastic elastic     8 Jun 22 06:27 index.latest
drwxr-xr-x 5 elastic elastic  4096 Jun 22 06:27 indices
-rw-r--r-- 1 elastic elastic 18430 Jun 22 06:27 meta-HTTdxw8bRRu2TsFDXJMLuA.dat
-rw-r--r-- 1 elastic elastic   386 Jun 22 06:27 snap-HTTdxw8bRRu2TsFDXJMLuA.dat
```
- Удалите индекс test и создайте индекс test-2. Приведите в ответе список индексов.
```
green open test-2 JG4exP78Ry2lz9B-nMyL3w 1 0 0 0 225b 225b
```
- Приведите в ответе запрос к API восстановления и итоговый список индексов.
```
# curl -k -u elastic -X POST "https://localhost:9200/_snapshot/netology_backup/backup1/_restore"
```
```
green open test-2 JG4exP78Ry2lz9B-nMyL3w 1 0 0 0 225b 225b
green open test   RmXjXNC1TCuAKZVuEUY1Kw 1 0 0 0 225b 225b
```
