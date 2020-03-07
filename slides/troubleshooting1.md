### Resolución de problemas

En esta sección, vamos a simular dos casos típicos de errores que ocurren en producción y veremos como
diagnosticarlos

^^^^^^

### Resolución de problemas: Caso 1

En este primer caso, vamos a ver cómo reacciona el maestro y la réplica ante la pérdida de conectividad.

^^^^^^

#### Resolución de problemas: Caso 1

⚠️ Antes de empezar: Levantar un maestro y una réplica de redis sin datos en memoria.

^^^^^^

#### Resolución de problemas: Caso 1

En el maestro, instalaremos el script [`fake2db`](https://github.com/emirozer/fake2db)

notes:

Este script ya lo tienes instalado en la máquina virtual si utilizas 
[nuestros ficheros Vagrant](https://github.com/Be-Core-Code/curso-redis-para-administradores-de-sistemas-vagrant)
para levantar las máquinas virtuales.

Si estás utilizando otras máquinas virtuales, sigue las instrucciones de instalación que se indican en el repositorio
de [`fake2db`](https://github.com/emirozer/fake2db) en github.

^^^^^^

#### Resolución de problemas: Caso 1

Ahora generaremos un conjunto de datos grande (1.000.000 de registros):

```bash 
(maestro) > fake2db --rows 200000 --db redis
```

⚠️ En mi máquina, este comando tardó casi 30 minutos en ejecutarse.


^^^^^^

#### Resolución de problemas: Caso 1

En el maestro, ejecutamos el siguiente comando:

```bash 
(master) > date; redis-cli debug sleep 90 ; date
Fri Mar  6 16:02:02 CET 2020
OK
Fri Mar  6 16:03:32 CET 2020
```

notes:

* Mostramos la fecha y hora antes de empezar
* Hacemos que el maestro no responda durante 90 segundos
* Volvemos a mostrar la hora

Mostramos la hora para ver el orden de los eventos en los logs del maestro y de la réplica.

^^^^^^

#### Resolución de problemas: Caso 1

Mirando el log de la réplica vemos lo siguiente:

```bash
4945:S 06 Mar 2020 16:02:56.611 # MASTER timeout: no data nor PING received...
4945:S 06 Mar 2020 16:02:56.611 # Connection with master lost.
4945:S 06 Mar 2020 16:02:56.611 * Caching the disconnected master state.
4945:S 06 Mar 2020 16:02:56.611 * Connecting to MASTER 192.168.158.10:6379
4945:S 06 Mar 2020 16:02:56.611 * MASTER <-> REPLICA sync started
4945:S 06 Mar 2020 16:02:56.612 * Non blocking connect for SYNC fired the event.
4945:S 06 Mar 2020 16:03:32.564 * Master replied to PING, replication can continue...
4945:S 06 Mar 2020 16:03:32.566 * Trying a partial resynchronization (request b10d220de03aff7705da31cc729368e66f072038:312488870).
4945:S 06 Mar 2020 16:03:32.566 * Successful partial resynchronization with master.
4945:S 06 Mar 2020 16:03:32.566 * MASTER <-> REPLICA sync: Master accepted a Partial Resynchronization.
```

^^^^^^

#### Resolución de problemas: Caso 1

* El maestro se pudo a dormir a las 16:02:02
* A las 16:02:56 (~60 segundos después) la réplica se da cuenta de que el maestro está caído. 
  Este tiempo viene marcado por el parámetro de configuración `repl-timeout`
* A las 16:03:32 (~30 segundos despúes) el maestro vuelve a la vida
* A las 16:03:32 la réplica se da cuenta de que el maestro está levantado (_Master replied to PING, replication can continue_)
  y comienza la sincronización de los datos
  
^^^^^^

#### Resolución de problemas: Caso 1

![master_slave_ping.001](/slides/images/master_slave_ping/master_slave_ping.001.jpeg)<!-- .element: style="height: 50vh" -->      
  
^^^^^^

#### Resolución de problemas: Caso 1

![master_slave_ping.002](/slides/images/master_slave_ping/master_slave_ping.002.jpeg)<!-- .element: style="height: 50vh" -->

^^^^^^

#### Resolución de problemas: Caso 1

![master_slave_ping.003](/slides/images/master_slave_ping/master_slave_ping.003.jpeg)<!-- .element: style="height: 50vh" -->

^^^^^^

#### Resolución de problemas: Caso 1

Por este motivo, en el ejemplo que hemos seguido antes la réplica tardó 60 segundos en darse cuenta
de que el maestro se había caído.

^^^^^^

#### Resolución de problemas: Caso 1

⚠️ Importante

En la documentación recomiendan que `repl-timeout` debe ser mayor que `repl-ping-replica-period`. 

Si no se hace, en ciertas condiciones de tráfico entre el maestro y la réplica, se pueden detectar timeouts cuando no los hay.      