### Comandos utilizados en este módulo

* [`INFO`](https://redis.io/commands/info)
* [`REPLICAOF`](https://redis.io/commands/replicaof): disponible a partir de la versión 5
* [`SLAVEOF`](https://redis.io/commands/slaveof) 

^^^^^^

### Opciones de configuración

* `slaveof` / `replicaof`: configura una instancia de redis como esclavo del maestro especificado
* `repl-backlog-size`: tamaño del backlog de replicación, por defecto 1 Mb
* `repl-timeout`
* `repl-ping-replica-period`: tiempo entre pings del maestro a las réplicas
* `client-output-buffer-limit`: tamaño del buffer de replicación

[Sobre `redis.conf`](https://redis.io/topics/config)