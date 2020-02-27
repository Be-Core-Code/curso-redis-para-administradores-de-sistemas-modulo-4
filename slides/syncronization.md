### Sincronización: `full resync`

¿Cómo ocurre el proceso de sincronización entre el maestro y la réplica?

^^^^^^

#### Sincronización: `full resync`

![](/slides/images/master_slaves/master_slaves.002.jpeg)<!-- .element: style="height: 50vh" -->

^^^^^^

#### Sincronización: `full resync`

![](/slides/images/master_slaves/master_slaves.003.jpeg)<!-- .element: style="height: 50vh" -->

^^^^^^

#### Sincronización: `full resync`

![](/slides/images/master_slaves/master_slaves.004.jpeg)<!-- .element: style="height: 50vh" -->

^^^^^^

#### Sincronización: `full resync`

![](/slides/images/master_slaves/master_slaves.005.jpeg)<!-- .element: style="height: 50vh" -->

^^^^^^

#### Sincronización: `full resync`

![](/slides/images/master_slaves/master_slaves.006.jpeg)<!-- .element: style="height: 50vh" -->

^^^^^^

#### Sincronización: `full resync`

Log de la réplica:

```bash
4614:S 27 Feb 2020 14:06:23.504 * Before turning into a replica, using my master parameters to synthesize a cached master: I may be able to synchronize with the new master with just a partial transfer.
4614:S 27 Feb 2020 14:06:23.504 * REPLICAOF 192.168.157.144:6379 enabled (user request from 'id=4 addr=192.168.157.146:58186 fd=9 name= age=3948 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=51 qbuf-free=32717 obl=0 oll=0 omem=0 events=r cmd=replicaof')
4614:S 27 Feb 2020 14:06:24.485 * Connecting to MASTER 192.168.157.144:6379
4614:S 27 Feb 2020 14:06:24.485 * MASTER <-> REPLICA sync started
4614:S 27 Feb 2020 14:06:24.486 * Non blocking connect for SYNC fired the event.
4614:S 27 Feb 2020 14:06:24.486 * Master replied to PING, replication can continue...
4614:S 27 Feb 2020 14:06:24.488 * Trying a partial resynchronization (request e9363a7b2620b0f20875eb98412f8c47b11a9b1a:6085).
4614:S 27 Feb 2020 14:06:24.488 * Full resync from master: bbea08c141a8dfd1cf68f1a3f6f66e1536663d4c:6084
4614:S 27 Feb 2020 14:06:24.488 * Discarding previously cached master state.
4614:S 27 Feb 2020 14:06:24.534 * MASTER <-> REPLICA sync: receiving 353 bytes from master
4614:S 27 Feb 2020 14:06:24.535 * MASTER <-> REPLICA sync: Flushing old data
4614:S 27 Feb 2020 14:06:24.535 * MASTER <-> REPLICA sync: Loading DB in memory
4614:S 27 Feb 2020 14:06:24.535 * MASTER <-> REPLICA sync: Finished with success
```

notes:

Vemos que la réplica, lo primero que intenta es hacer una sincronización parcial. Al
no poder llevarla a cabo, comienza una sincronización completa.

^^^^^^

#### Sincronización: `full resync`

Log del maestro:

```bash
4437:M 27 Feb 2020 14:06:24.487 * Replica 192.168.157.146:6379 asks for synchronization
4437:M 27 Feb 2020 14:06:24.487 * Partial resynchronization not accepted: Replication ID mismatch (Replica asked for 'e9363a7b2620b0f20875eb98412f8c47b11a9b1a', m
4437:M 27 Feb 2020 14:06:24.487 * Starting BGSAVE for SYNC with target: disk
4437:M 27 Feb 2020 14:06:24.487 * Background saving started by pid 4562
4562:C 27 Feb 2020 14:06:24.489 * DB saved on disk
4562:C 27 Feb 2020 14:06:24.489 * RDB: 2 MB of memory used by copy-on-write
4437:M 27 Feb 2020 14:06:24.533 * Background saving terminated with success
4437:M 27 Feb 2020 14:06:24.533 * Synchronization with replica 192.168.157.146:6379 succeeded
```

^^^^^^

### Sincronización: `partial resync`

El proceso `full resync` es un proceso costoso ya que requiera que se transfiera un snapshot completo
entre el maestro y la réplica.

Redis facilita un proceso de sincronización parcial que permite que sólo se transfieran los comandos
que están pendientes de aplicar en la réplica y no todos los datos.


^^^^^^

### Sincronización: `partial resync`

Para hacerlo, redis dispone de dos parámetros:

* `master_replid`: es el ID de replicación del maestro
* `master_repl_offset`: offset del stream de comandos de sincronización

^^^^^^

#### Sincronización: `partial resync`

A medida que el maestro recibe comandos para modificar los datos, este genera un stream 
con ellos que envía a las réplicas. 

^^^^^^

#### Sincronización: `partial resync`

A medida que va enviando los comandos a las réplicas, va ajustando el parámetro `master_repl_offset` para saber
qué es lo que ha enviado ya. 

^^^^^^

#### Sincronización: `partial resync`

Cuando la réplica recibe las instrucciones, actualiza el parámetro `master_repl_offset` para _apuntar_ hasta qué punto 
del stream del maestro ha recibido.

^^^^^^

#### Sincronización: `partial resync`

Si todo está sincronizado, el parámetro `master_repl_offset` debe contener el mismo valor 
en el maestro y en las réplicas.

```redis-cli
redis-cli (maestro) > INFO Replication
...
master_replid:bbea08c141a8dfd1cf68f1a3f6f66e1536663d4c
master_repl_offset:17226
...  
```

```redis-cli
redis-cli (replica1) > INFO Replication
...
master_replid:bbea08c141a8dfd1cf68f1a3f6f66e1536663d4c
master_repl_offset:17226
...
  
```

^^^^^^

#### 💻️ Sincronización: `partial resync`

* Vamos a desactivar la red en la máquina virtual de la réplica

```bash
(replica1) > rc-service networking stop
```

^^^^^^

#### 💻️ Sincronización: `partial resync`

* En el maestro crearemos una nueva clave

```redis-cli
redis-cli (maestro) > HSET curso:4 nombre "Curso de introducción a docker" duration 30 
```

* Miramos cómo ha cambiado el valor de `master_repl_offset`

```redis-cli
redis-cli (maestro) > INFO Replication
...
master_replid:bbea08c141a8dfd1cf68f1a3f6f66e1536663d4c
master_repl_offset:17787
...  
```

^^^^^^

#### 💻️ Sincronización: `partial resync`

* Levantamos la red en la máquina virtual de la réplica

```bash
(replica1) > rc-service networking start
```

* Verificamos el valor de `master_repl_offset`

```redis-cli
redis-cli (replica1) > INFO Replication
...
master_replid:bbea08c141a8dfd1cf68f1a3f6f66e1536663d4c
master_repl_offset:17885
...  
``` 

notes:

Aunque el valor no es el mismo (en el maestro el valor era 17787) **el valor de `master_repl_offset` es
mayor que el que teníamos antes de que la red se parase**.

Que no coincidan sólo nos dice que cuando preparé las diapositivas, mientras levanté la red en la réplica
y me conecté con el cliente de Redis y ejecuté el comando [`INFO`](https://redis.io/commands/info)
el valor de `master_repl_offset` aumentó.

^^^^^^

#### 💻️ Sincronización: `partial resync`

* Log del maestro:

```bash 
4437:M 27 Feb 2020 16:34:06.157 * Replica 192.168.157.146:6379 asks for synchronization
4437:M 27 Feb 2020 16:34:06.157 * Partial resynchronization request from 192.168.157.146:6379 accepted. Sending 183 bytes of backlog starting from offset 17605.
```

* Log de la réplica:

```bash
4614:S 27 Feb 2020 16:33:38.971 * MASTER <-> REPLICA sync started
4614:S 27 Feb 2020 16:33:38.971 * Non blocking connect for SYNC fired the event.
4614:S 27 Feb 2020 16:33:38.971 * Master replied to PING, replication can continue...
4614:S 27 Feb 2020 16:33:38.972 * Trying a partial resynchronization (request bbea08c141a8dfd1cf68f1a3f6f66e1536663d4c:17605).
4614:S 27 Feb 2020 16:33:38.972 * Successful partial resynchronization with master.
4614:S 27 Feb 2020 16:33:38.972 * MASTER <-> REPLICA sync: Master accepted a Partial Resynchronization.
4614:S 27 Feb 2020 16:33:39.071 * 1 changes in 900 seconds. Saving...
4614:S 27 Feb 2020 16:33:39.072 * Background saving started by pid 5024
5024:C 27 Feb 2020 16:33:39.074 * DB saved on disk
5024:C 27 Feb 2020 16:33:39.074 * RDB: 2 MB of memory used by copy-on-write
4614:S 27 Feb 2020 16:33:39.173 * Background saving terminated with success 
```
