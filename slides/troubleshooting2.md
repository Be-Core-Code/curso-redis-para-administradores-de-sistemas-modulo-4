### Resolución de problemas: Caso 2

En este caso vamos a simular una condición de carrera que se puede dar en producción:

El maestro recibe un volumen de escrituras tan alto que la réplica no puede hacer un _full resync_.

^^^^^^

#### 💻️ Resolución de problemas: Caso 2

Punto de partida: Un maestro y una réplica con 2.000.000 de datos cargados.

![troubleshooting2.001.jpeg](/slides/images/troubleshooting2/troubleshooting2.001.jpeg)<!-- .element: style="height: 50vh" -->
 
notes:

Podéis generar esos dos millones de claves usando el script [`redis-random-data-generator`](https://github.com/SaminOz/redis-random-data-generator):

```bash
(maestro) > node /usr/local/redis-random-data-generator hash 2000000 session
``` 


^^^^^^

#### 💻️ Resolución de problemas: Caso 2

Para simular la situación que se daría en producción, necesitaremos dos herramientas:

* [`redis-random-data-generator`](https://github.com/SaminOz/redis-random-data-generator): lo utilizaremos para
  simular las escrituras en redis
* `tc`: para ralentizar la comunicación entre maestro y réplica 
* Reducir el buffer: `config set client-output-buffer-limit 'slave 134217728 33554432 30'`

notes:

El motivo por el que vamos a ralentizar la comunicación entre maestro y réplica es para aumentar el tiempo
que se tarda en transferir el fichero `dump.rdb` entre el maestro y la réplica. Otra forma de hacerlo
sería crear varios cientos de megas o gigas de datos en memoria. Utilizamos la primera opción
por ser más cómoda y ágila la hora de hacer el laboratorio. 

La herramienta [`redis-random-data-generator`](https://github.com/SaminOz/redis-random-data-generator) y el comando
`tc` están ya instalados en la máquina virtual si has utilizado 
nuestro [Vagrantfile](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas-vagrant) para generar las máquinas virtuales.

Si estás utilizando tus propias máquinas virtuales:

* Clona el repositorio de [`redis-random-data-generator`](https://github.com/SaminOz/redis-random-data-generator)

```bash
> git clone  https://github.com/SaminOz/redis-random-data-generator /usr/local/redis-random-data-generator
``` 
 
* Instala el paquete `iproute2` en la distribución que hayas instalado en tu máquina virtual     
 
 
^^^^^^

#### 💻️ Resolución de problemas: Caso 2

En el maestro, ralentizamos la transferencia de red:

```bash
(maestro) > tc qdisc add dev eth1 root netem delay 1000ms
```

^^^^^^

#### 💻️ Resolución de problemas: Caso 2


Hacemos que la réplica deje de serlo y limpiamos sus datos:
```redis-cli
redis-cli (replica) > REPLICAOF NO ONE
OK
redis-cli (replica) > FLUSHALL
OK 
```

^^^^^^

#### 💻️ Resolución de problemas: Caso 2

![troubleshooting2.002.jpeg](/slides/images/troubleshooting2/troubleshooting2.002.jpeg)<!-- .element: style="height: 50vh" -->


^^^^^^

#### 💻️ Resolución de problemas: Caso 2

Ahora haremos lo siguente:

* Volveremos a hacer que la réplica sea réplica del maestro ejecutando el comando [`REPLICAOF`](https://redis.io/commands/replicaof)

```redis-cli
redis-cli (replica) > REPLICAOF 192.168.158.10
OK
```

* **Antes de que termine de generarse el fichero dump.rdb** empezaremos a cargar en el maestro un 
millón de claves nuevas

```bash
(maestro) > node /usr/local/redis-random-data-generator hash 1000000 session
``` 


^^^^^^

#### 💻️ Resolución de problemas: Caso 2

En este momento se comenzará un _full resync_ y el maestro comenzará a crear un fichero `dump.rb`:

```bash 
# Maestro
4434:M 07 Mar 2020 13:39:36.064 * Replica 192.168.158.11:6379 asks for synchronization
4434:M 07 Mar 2020 13:39:36.064 * Partial resynchronization not accepted: Replication ID mismatch (Replica asked for '5fb70724a00f9e3355328776edefe97575fafeca', my replication IDs are '11009e200e8022dc660c1a10042909b6907d974b' and '0000000000000000000000000000000000000000')
4434:M 07 Mar 2020 13:39:36.064 * Starting BGSAVE for SYNC with target: disk
4434:M 07 Mar 2020 13:39:36.081 * Background saving started by pid 4720
```

```bash 
# Réplica
4440:S 07 Mar 2020 13:39:35.169 * Before turning into a replica, using my master parameters to synthesize a cached master: I may be able to synchronize with the new master with just a partial transfer.
4440:S 07 Mar 2020 13:39:35.169 * REPLICAOF 192.168.158.10:6379 enabled (user request from 'id=3 addr=127.0.0.1:47750 fd=9 name= age=669 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=50 qbuf-free=32718 obl=0 oll=0 omem=0 events=r cmd=replicaof')
4440:S 07 Mar 2020 13:39:36.061 * Connecting to MASTER 192.168.158.10:6379
4440:S 07 Mar 2020 13:39:36.061 * MASTER <-> REPLICA sync started
4440:S 07 Mar 2020 13:39:36.061 * Non blocking connect for SYNC fired the event.
4440:S 07 Mar 2020 13:39:36.062 * Master replied to PING, replication can continue...
4440:S 07 Mar 2020 13:39:36.063 * Trying a partial resynchronization (request 5fb70724a00f9e3355328776edefe97575fafeca:886037448).
4440:S 07 Mar 2020 13:39:36.081 * Full resync from master: 11009e200e8022dc660c1a10042909b6907d974b:886037406
4440:S 07 Mar 2020 13:39:36.081 * Discarding previously cached master state.
```

^^^^^^

#### 💻️ Resolución de problemas: Caso 2

Dado que el maestro está recibiendo operaciones de escritura, el buffer de replicación comienza a 
llenarse con estos comandos nuevos.

**A la vez, las escrituras se van ejecutando en el maestro por lo que los datos quedan en memoria.**

^^^^^^

#### 💻️ Resolución de problemas: Caso 2

![troubleshooting2.003.jpeg](/slides/images/troubleshooting2/troubleshooting2.003.jpeg)<!-- .element: style="height: 50vh" -->

^^^^^^

#### 💻️ Resolución de problemas: Caso 2

Como el buffer es muy pequeño (lo hemos limitado a unos 32MB para este laboratorio) se llena y el maestro 
desconecta a la réplica 13:40:17. 

```bash
# Maestro
4434:M 07 Mar 2020 13:40:17.452 # Client id=16 addr=192.168.158.11:34400 fd=9 name= age=41 idle=41 flags=S db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=16351 oll=8173 omem=134233352 events=r cmd=psync scheduled to be closed ASAP for overcoming of output buffer limits.
4434:M 07 Mar 2020 13:40:17.619 # Connection with replica 192.168.158.11:6379 lost.
4434:M 07 Mar 2020 13:40:18.559 * Replica 192.168.158.11:6379 asks for synchronization
4434:M 07 Mar 2020 13:40:18.559 * Full resync requested by replica 192.168.158.11:6379
4434:M 07 Mar 2020 13:40:18.559 * Can't attach the replica to the current BGSAVE. Waiting for next BGSAVE for SYNC
```

```bash
# Réplica
4440:S 07 Mar 2020 13:40:17.963 # I/O error reading bulk count from MASTER: Resource temporarily unavailable
4440:S 07 Mar 2020 13:40:18.521 * Connecting to MASTER 192.168.158.10:6379
4440:S 07 Mar 2020 13:40:18.521 * MASTER <-> REPLICA sync started
4440:S 07 Mar 2020 13:40:18.521 * Non blocking connect for SYNC fired the event.
4440:S 07 Mar 2020 13:40:18.532 * Master replied to PING, replication can continue...
4440:S 07 Mar 2020 13:40:18.548 * Partial resynchronization not possible (no cached master)
```

^^^^^^

#### 💻️ Resolución de problemas: Caso 2

![troubleshooting2.004.jpeg](/slides/images/troubleshooting2/troubleshooting2.004.jpeg)<!-- .element: style="height: 50vh" -->

^^^^^^

#### 💻️ Resolución de problemas: Caso 2

Pasados unos segundos, cuando ha terminado el anterior [`BGSAVE`](https://redis.io/commands/bgsave) 
el maestro comienza un nuevo _full resync_
y vuelve a lanzar un [`BGSAVE`](https://redis.io/commands/bgsave) (13:41:09)

```bash
# Maestro
4720:C 07 Mar 2020 13:41:09.773 * DB saved on disk
4720:C 07 Mar 2020 13:41:09.793 * RDB: 76 MB of memory used by copy-on-write
4434:M 07 Mar 2020 13:41:09.865 * Background saving terminated with success
4434:M 07 Mar 2020 13:41:09.865 * Starting BGSAVE for SYNC with target: disk
4434:M 07 Mar 2020 13:41:09.880 * Background saving started by pid 4735
```

```bash
# Réplica
4440:S 07 Mar 2020 13:41:09.881 * Full resync from master: 11009e200e8022dc660c1a10042909b6907d974b:1108892332
```

^^^^^^

#### 💻️ Resolución de problemas: Caso 2

![troubleshooting2.005.jpeg](/slides/images/troubleshooting2/troubleshooting2.005.jpeg)<!-- .element: style="height: 50vh" -->

^^^^^^
#### 💻️ Resolución de problemas: Caso 2

...y de nuevo vuelve a pasar lo mismo ya que siguen llegando escrituras al maestro y el buffer se vuelve a llenar
antes de poder finalizar el _full resync_:

```bash
# Maestro
4434:M 07 Mar 2020 13:41:58.001 # Client id=18 addr=192.168.158.11:34402 fd=9 name= age=100 idle=100 flags=S db=0 sub=0 psub=0 multi=-1 qbuf=0 qbuf-free=0 obl=16343 oll=7854 omem=128994096 events=r cmd=psync scheduled to be closed ASAP for overcoming of output buffer limits.
4434:M 07 Mar 2020 13:41:58.059 # Connection with replica 192.168.158.11:6379 lost.
4434:M 07 Mar 2020 13:41:59.646 * Replica 192.168.158.11:6379 asks for synchronization
4434:M 07 Mar 2020 13:41:59.646 * Full resync requested by replica 192.168.158.11:6379
4434:M 07 Mar 2020 13:41:59.646 * Can't attach the replica to the current BGSAVE. Waiting for next BGSAVE for SYNC
```

```bash
# Réplica
4440:S 07 Mar 2020 13:41:59.385 # I/O error reading bulk count from MASTER: Resource temporarily unavailable
4440:S 07 Mar 2020 13:41:59.606 * Connecting to MASTER 192.168.158.10:6379
4440:S 07 Mar 2020 13:41:59.606 * MASTER <-> REPLICA sync started
4440:S 07 Mar 2020 13:41:59.606 * Non blocking connect for SYNC fired the event.
4440:S 07 Mar 2020 13:41:59.618 * Master replied to PING, replication can continue...
4440:S 07 Mar 2020 13:41:59.635 * Partial resynchronization not possible (no cached master)
```

^^^^^^

#### 💻️ Resolución de problemas: Caso 2 

Solución: Ampliar el tamaño del buffer para que pueda finalizarse el proceso de _full resync_: 

```bash
config set client-output-buffer-limit 'slave 536870912 134217728 120'
```