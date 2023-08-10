<div align="center"><h2> 2. Настройка Consul </h2></div>

***
### Немного теории:
  * В концепции кластера все значения/параметры/статусы/прочее должны храниться в единном месте - `DCS (Distributed Config Store)` и должны быть одинаковыми для всех участников кластера. Другими словами - это единное хранилище вида "Ключ-Значение". 
  * PostgreSQL не умеет напрямую работать с `DCS (Distributed Config Store)` поэтому ему нужно некоторое ПО, в нашем случае - это [`Patroni`](https://github.com/zalando/patroni). Но о нем позже.
  * В качестве `DCS (Distributed Config Store)` для [`Patroni`](https://github.com/zalando/patroni) могут выступать:
    * [`Consul`](https://github.com/hashicorp/consul). ❗️Именно его мы и будет использовать
    * [`etcd`](https://github.com/etcd-io/etcd)
    * [`ZooKeeper`](https://zookeeper.apache.org/)
  
***

