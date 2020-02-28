### Comandos utilizados en este módulo

* [`SLAVEOF`](https://redis.io/commands/slaveof) 
* [`REPLICAOF`](https://redis.io/commands/replicaof): disponible a partir de la versión 5
^^^^^^

### Opciones de configuración

* `slaveof` / `replicaof`: configura una instancia de redis como esclavo del maestro especificado
* `repl-backlog-size`: tamaño del backlog de replicación, por defecto 1 Mb

[Sobre `redis.conf`](https://redis.io/topics/config)