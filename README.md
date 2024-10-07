# Charts

## Kafka + Zookeeper

ZooKeeper будет официально **deprecated** в Apache Kafka с выходом версии 4.0, которая запланирована на 2024 год. Тем не менее, для быстрого переезда на новую инфраструктуру считаю адекватным решением поднять кластер из 3 ZK и 3 Kafka, чтобы текущие приложения разработчиков могли сразу работать в новых условиях.

- **Storage Class** сейчас генерируется по умолчанию.
- Пароли пока что всюду `changeit`, нужно будет изменить.
- PVC для сохранения данных есть.

### Изменения в `charts/kafka/values.yaml`

- Брокеры сейчас имеют тип **NodePort** и `nodePorts`, а также `externalIPs` из моего Minikube.

### Изменения в `charts/kafka/charts/zookeeper/values.yaml`

- Раскомментировать `affinity` и добавить метки узлов.
- Добавить `nodeSelector` с метками узлов с зоной доступности согласно таковым в кластере.

### Настройка SSL авторизации

Файл `values-v2-auth.yaml` нужен для настройки SSL авторизации между ZK и Kafka, генерация ключей осуществляется скриптами из папки `tls`.

## Redis

Стандартный кластер с мастером, 3 репликами и 3 Sentinel. Отказоустойчивость обеспечивается за счёт **podAntiAffinity**.

## Установка Kafka + Zookeeper + Redis

(когда-нибудь тут будет `helm repo add kf-zk-rd  https://gitlab.xplanet.tech/devops/charts.git`, но пока что так:)

   ```bash

   git clone git@gitlab.xplanet.tech:devops/charts.git
   cd charts
   helm dependency update ./kafka-zk-redis
   cd kafka-zk-redis/charts
   tar -xzf kafka-*.tgz
   tar -xzf redis-*.tgz

   helm install release-name ./kafka-zk-redis -n namespace -f kafka-zk-redis/values-v1.yaml
   helm upgrade  dev ./kafka-zk-redis -n namespace -f kafka-zk-redis/values-v1.yaml --set global.redis.password=changeit --render-subchart-notes

а в перспективе:

   helmfile apply