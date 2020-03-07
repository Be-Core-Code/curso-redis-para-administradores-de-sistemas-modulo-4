### Ejercicio 1: 

Añade una segunda réplica al cluster

### Ejercicio 2: 

Utilizando el parámetro de configuración `requirepass`, configura el cluster para que las réplicas se conecten al maestro
usando contraseñas.

notes:

Si nuestra instancia de redis está conectada a internet, un atacante podría lanzar un ataque de fuerza bruta para
intentar conectarse al maestro. Dado que Redis es tan rápido, el ratio al que el atacante prueba contraseñas puede
ser de varios miles o cientos de miles por segundo, por lo que es importante **utilizar una contraseña muy fuerte** si
estamos en esta situación.

Redis no incluye ningún sistema de protección contra ataques de fuerza bruta.