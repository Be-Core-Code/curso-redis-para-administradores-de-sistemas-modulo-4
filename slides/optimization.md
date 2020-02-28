### Optimización de la replicación

Como hemos podido ver en las secciones anteriores, **configurar la replicación en Redis
es muy sencillo** 😎

^^^^^^

#### Optimización de la replicación

Hay sin embargo varios parámetros para configurar el rendimiento del proceso de sincronización
que requieren atención.

^^^^^^

#### 💻️ Optimización de la replicación

* Utilizando el script 
  [`prepara_datos_para_replicar.sh`](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas/blob/master/modulo-4/prepara_datos_para_replicar.sh)
  vamos a generar un fichero con un conjunto de datos que llamaremos `replicacion.data`:

```bash
(maestro) > ./prepara_datos_para_replicar.sh 1000
unix2dos: converting file replicacion.data to DOS format...
Generadas 1000 claves de tipo hash en el fichero replicacion.data 
``` 

^^^^^^

#### 💻️ Optimización de la replicación

* Limpiamos los datos del maestro usando el comando [`FLUSHALL`](https://redis.io/commands/flushall)

```redis-cli
redis-cli (maestro) > FLUSHALL 
OK
```

^^^^^^

#### 💻️ Optimización de la replicación

* Verificamos que la réplica tampoco tiene ningún dato

```redis-cli
redis-cli (replica1) > KEYS *
(empty list or set)
```

^^^^^^

#### 💻️ Optimización de la replicación

* Apagamos la red en la réplica de forma que no pueda sincronizarse con el maestro

```bash 
(replica1) > rc-service networking stop
```

^^^^^^

#### 💻️ Optimización de la replicación

* Generamos 1000 claves en el maestro

```bash
(maestro) > ./prepara_datos_para_replicar.sh 1000
unix2dos: converting file replicacion.data to DOS format...
Generadas 1000 claves de tipo hash en el fichero replicacion.data 
``` 

* ...y las cargamos mientras la réplica está sin conexión de red

```bash 
(maestro) > cat replicacion.data | redis-cli --pipe
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 1000
```

^^^^^^

#### 💻️ Optimización de la replicación

* Activamos de nuevo la red en la réplica

```bash 
(replica1) > rc-service networking start
```

^^^^^^

#### 💻️ Optimización de la replicación

* Si miramos el log del maestro:

```bash
4437:M 27 Feb 2020 18:15:11.598 * Replica 192.168.157.146:6379 asks for synchronization
4437:M 27 Feb 2020 18:15:11.598 * Partial resynchronization request from 192.168.157.146:6379 accepted. Sending 130112 bytes of backlog starting from offset 545584.
``` 

* Si miramos el log de la réplica:

```bash
4614:S 27 Feb 2020 18:06:57.835 * MASTER <-> REPLICA sync started
4614:S 27 Feb 2020 18:06:57.836 * Non blocking connect for SYNC fired the event.
4614:S 27 Feb 2020 18:06:57.837 * Master replied to PING, replication can continue...
4614:S 27 Feb 2020 18:06:57.838 * Trying a partial resynchronization (request bbea08c141a8dfd1cf68f1a3f6f66e1536663d4c:545584).
4614:S 27 Feb 2020 18:06:57.839 * Successful partial resynchronization with master.
4614:S 27 Feb 2020 18:06:57.839 * MASTER <-> REPLICA sync: Master accepted a Partial Resynchronization.
```

**Como esperábamos, se ha producido un `partial resync`**

^^^^^^

#### 💻️ Optimización de la replicación

* Desactivamos de nuevo la red en la réplica

```bash 
(replica1) > rc-service networking stop
```

^^^^^^

#### 💻️ Optimización de la replicación

* Limpiamos los datos del maestro usando el comando [`FLUSHALL`](https://redis.io/commands/flushall)

```redis-cli
redis-cli (maestro) > FLUSHALL 
OK
```

^^^^^^

#### 💻️ Optimización de la replicación

* Verificamos que la réplica tampoco tiene ningún dato

```redis-cli
redis-cli (replica1) > KEYS *
(empty list or set)
```

^^^^^^

#### 💻️ Optimización de la replicación

* Apagamos de nuevo la red en la réplica

```bash 
(replica1) > rc-service networking stop
```

^^^^^^

#### 💻️ Optimización de la replicación

* Generamos en este caso **11000 claves en el maestro**

```bash
(maestro) > ./prepara_datos_para_replicar.sh 11000
unix2dos: converting file replicacion.data to DOS format...
Generadas 11000 claves de tipo hash en el fichero replicacion.data 
``` 

* ...y las cargamos mientras la réplica está sin conexión de red

```bash 
(maestro) > cat replicacion.data | redis-cli --pipe
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 11000
```

^^^^^^

#### 💻️ Optimización de la replicación

* Activamos de nuevo la red en la réplica

```bash 
(replica1) > rc-service networking start
```

^^^^^^

#### 💻️ Optimización de la replicación

* Si miramos el log del maestro:

```bash
4437:M 27 Feb 2020 18:29:48.542 # Connection with replica 192.168.157.146:6379 lost.
4437:M 27 Feb 2020 18:29:48.543 * Replica 192.168.157.146:6379 asks for synchronization
4437:M 27 Feb 2020 18:29:48.543 * Unable to partial resync with replica 192.168.157.146:6379 for lack of backlog (Replica request was: 2117792).
4437:M 27 Feb 2020 18:29:48.543 * Starting BGSAVE for SYNC with target: disk
4437:M 27 Feb 2020 18:29:48.543 * Background saving started by pid 36948
36948:C 27 Feb 2020 18:29:48.556 * DB saved on disk
36948:C 27 Feb 2020 18:29:48.556 * RDB: 0 MB of memory used by copy-on-write
4437:M 27 Feb 2020 18:29:48.639 * Background saving terminated with success
4437:M 27 Feb 2020 18:29:48.643 * Synchronization with replica 192.168.157.146:6379 succeeded
``` 

* Si miramos el log de la réplica:

```bash
4614:S 27 Feb 2020 18:20:27.325 * MASTER <-> REPLICA sync started
4614:S 27 Feb 2020 18:20:27.325 * Non blocking connect for SYNC fired the event.
4614:S 27 Feb 2020 18:20:27.325 * Master replied to PING, replication can continue...
4614:S 27 Feb 2020 18:20:27.326 * Trying a partial resynchronization (request bbea08c141a8dfd1cf68f1a3f6f66e1536663d4c:2117792).
4614:S 27 Feb 2020 18:20:27.327 * Full resync from master: bbea08c141a8dfd1cf68f1a3f6f66e1536663d4c:3558875
4614:S 27 Feb 2020 18:20:27.327 * Discarding previously cached master state.
4614:S 27 Feb 2020 18:20:27.415 * MASTER <-> REPLICA sync: receiving 924184 bytes from master
4614:S 27 Feb 2020 18:20:27.420 * MASTER <-> REPLICA sync: Flushing old data
4614:S 27 Feb 2020 18:20:27.420 * MASTER <-> REPLICA sync: Loading DB in memory
4614:S 27 Feb 2020 18:20:27.431 * MASTER <-> REPLICA sync: Finished with success
```


^^^^^^

#### 💻️ Optimización de la replicación

**Se ha producido un `full resync` 😱**

![wtf](/slides/images/wtf.jpg)<!-- .element: style="height: 55vh"-->

^^^^^^

#### Optimización de la replicación

El nodo maesto tiene un _Replication backlog_ en el que se almacenan los comandos  mientras alguna de las
réplicas está caída.

Cuando estas réplicas se levantan, el maestro compara el último offset de la réplica y extrae del backlog
los comandos que le tiene que enviar.

^^^^^^

#### Optimización de la replicación

¿Qué nos ha ocurrido en el segundo caso del ejemplo anterior? Que los comandos de redis no cabían en el backlog
y por lo tanto **era necesario hacer una sincronización completa**

^^^^^^

#### Optimización de la replicación

El parámetro `repl-backlog-size` (1mb por defecto) es el que controla el tamaño del backlog

Vamos a aumentar el valor de este parámetro a 1.5mb

^^^^^^

#### 💻️ Optimización de la replicación

* Limpiamos los datos del maestro usando el comando [`FLUSHALL`](https://redis.io/commands/flushall)

```redis-cli
redis-cli (maestro) > FLUSHALL 
OK
```

^^^^^^

#### 💻️ Optimización de la replicación

* Verificamos que la réplica tampoco tiene ningún dato

```redis-cli
redis-cli (replica1) > KEYS *
(empty list or set)
```

^^^^^^

#### 💻️ Optimización de la replicación

* Ampliamos el tamaño del backlog en el maestro

```redis-cli
redis-cli (maestro) > CONFIG SET repl-backlog-size 1500kb
OK 
```

notes:

Para persistir este cambio, debemos incluirlo en el fichero `/etc/redis.conf`

^^^^^^

#### 💻️ Optimización de la replicación

* Apagamos de nuevo la red en la réplica

```bash 
(replica1) > rc-service networking stop
```

^^^^^^

#### 💻️ Optimización de la replicación

* Generamos en este caso **11000 claves en el maestro**

```bash
(maestro) > ./prepara_datos_para_replicar.sh 11000
unix2dos: converting file replicacion.data to DOS format...
Generadas 11000 claves de tipo hash en el fichero replicacion.data 
``` 

* ...y las cargamos mientras la réplica está sin conexión de red

```bash 
(maestro) > cat replicacion.data | redis-cli --pipe
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 11000
```

^^^^^^

#### 💻️ Optimización de la replicación

* Activamos de nuevo la red en la réplica

```bash 
(replica1) > rc-service networking start
```

^^^^^^

#### 💻️ Optimización de la replicación

* Si miramos el log del maestro:

```bash
4431:M 28 Feb 2020 09:21:28.846 * Background saving started by pid 4524
4524:C 28 Feb 2020 09:21:28.853 * DB saved on disk
4524:C 28 Feb 2020 09:21:28.853 * RDB: 0 MB of memory used by copy-on-write
4431:M 28 Feb 2020 09:21:28.947 * Background saving terminated with success
4431:M 28 Feb 2020 09:21:54.373 * Replica 192.168.157.147:6379 asks for synchronization
4431:M 28 Feb 2020 09:21:54.386 * Partial resynchronization request from 192.168.157.147:6379 accepted. Sending 1441107 bytes of backlog starting from offset 351.
```

* Si miramos el log dela réplica:

```bash
4599:S 28 Feb 2020 09:21:54.371 * Connecting to MASTER 192.168.157.144:6379
4599:S 28 Feb 2020 09:21:54.371 * MASTER <-> REPLICA sync started
4599:S 28 Feb 2020 09:21:54.372 * Non blocking connect for SYNC fired the event.
4599:S 28 Feb 2020 09:21:54.372 * Master replied to PING, replication can continue...
4599:S 28 Feb 2020 09:21:54.374 * Trying a partial resynchronization (request 15f1d6577911fa2dc1a1fb7ef73f5add8ca79f8f:351).
4599:S 28 Feb 2020 09:21:54.374 * Successful partial resynchronization with master.
```

^^^^^^

#### 💻️ Optimización de la replicación

**Se ha producido un `partial resync` 😎**

![carlton_dance](/slides/images/carlton_dance.gif)
