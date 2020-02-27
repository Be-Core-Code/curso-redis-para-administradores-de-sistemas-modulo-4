### Configuración

Redis dispone de varias opciones de configuración para controlar el comportamiento del nodo esclavo.

La única realmente necesaria para configurarlo es `replicaof` o usando el comando 
[`REPLICAOF`](https://redis.io/commands/replicaof).

^^^^^^

#### 💻️Configuración

Vamos a empezar levantando un maestro y un esclavo.

![master_slave_configuration](/slides/images/master_slaves/master_slaves.001.jpeg)<!-- .element: style="height: 40vh" -->

^^^^^^

#### 💻️ Configuración

* Lo primero que necesitamos hacer es levantar dos máquinas virtuales

notes:

Puedes duplicar la máquina del curso y cambiarle de nombre.

Para cambiarle de nombre debes seguir los siguientes pasos:

* Duplicar la máquina virtual
* Levantar la máquina virtual y acceder a ella
* Editar el fichero `/etc/hostname` y cambiar el nombre a `esclavo1`
* Editar el fichero `/etc/hosts` y cambiarla para que contenga:

```bash
127.0.0.1	esclavo1.localdomain esclavo1 localhost.localdomain localhost
::1		localhost localhost.localdomain
```
* Reiniciar la máquina 

^^^^^^

#### 💻️ Configuración

* Accedemos al esclavo y ejecutamos el siguiente comando:

```redis-cli
redis-cli (esclavo1) > REPLICAOF 192.168.157.144 6379
OK
```

^^^^^^

#### 💻️ Configuración

* Accedemos al maestro y vemos la configuración:

```redis-cli
redis-cli (maestro) > INFO Replication 
# Replication
role:master                                                       <----------
connected_slaves:1                                                <----------
slave0:ip=192.168.157.146,port=6379,state=online,offset=266,lag=0 <----------
master_replid:bbea08c141a8dfd1cf68f1a3f6f66e1536663d4c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:266
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:266
```

^^^^^^

#### 💻️ Configuración

* Accedemos al esclavo y vemos la configuración:

```redis-cli
redis-cli (esclavo) > INFO Replication
# Replication
role:slave                                                       <----------
master_host:192.168.157.144                                      <----------
master_port:6379                                                 <----------
master_link_status:up
master_last_io_seconds_ago:10
master_sync_in_progress:0
slave_repl_offset:84
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:bbea08c141a8dfd1cf68f1a3f6f66e1536663d4c           <----------
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:84
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:84 
```

^^^^^^

#### 💻️ Configuración

* Limpiamos los datos en el maestro con el comando [`FLUSHALL`](https://redis.io/commands/flushall)

```redis-cli
redis-cli (maestro) > FLUSHALL
OK 
```

^^^^^^

#### 💻️ Configuración

* Creamos dos claves en el maestro

```redis-cli
redis-cli (maestro) > HSET curso:1 nombre "curso de redis para administradores de sistemas" duracion 25
(integer) 1
redis-cli (maestro) > HSET curso:2 nombre: "curso de git avanzado" duracion 30
(integer) 1    
```


^^^^^^

#### 💻️ Configuración

* Si ahora vamos al esclavo y listamos las claves:

```redis-cli
redis-cli (esclavo1) > KEYS *
1) "curso:2"
2) "curso:1"
redis-cli (esclavo1) > HGETALL curso:2
1) "nombre:"
2) "curso de git avanzado"
3) "duracion"
4) "30" 
```

^^^^^^

#### 💻️ Configuración

* Para persistir los cambios en la configuración del esclavo, editamos el fichero
  `/etc/redis.conf` y añadimos la línea:
  
```bash
# /etc/resolv.conf 
slaveof 192.168.157.144 6789 
```   

^^^^^^

#### 💻️ Configuración

* Podemos utilizar el comando `REPLICAOF NO ONE` para que un nodo deje de replicar a otro

^^^^^^


### Documentación
[https://redis.io/topics/replication](https://redis.io/topics/replication)