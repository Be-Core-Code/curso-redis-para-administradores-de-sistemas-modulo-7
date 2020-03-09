### Configuración de conexiones de clientes

A continuación describimos varios parámetros de configuración que afectan a los clientes
que se conectan a Redis

^^^^^^

### Clientes: `timeout`

Cuando una conexión de un cliente no se utiliza durante los segundos indicados por este parámetro,
el servidor de redis la cierra.

```bash
# /etc/redis.conf
timeout 0
```

El valor 0 deshabilita el timeout.


^^^^^^

### Clientes: `tcp-backlog`

Este parámetro es el que se pasa por defecto marca el tamaño del backlog que se pasa a la función `listen()` del socket.

Este backlog indica el tamaño de la cola de conexciones que se mantienen en espera antes de 
empezar a tirar (`DROP`) las conexiones nuevas.

^^^^^^

#### Clientes: `tcp-backlog`

**El kernel de Linux trunca este valor al valor de `net.core.somaxconn`.**

Por eso, como comentamos en la sección [Configuración de Linux](/#linux_configuration),
si aumentamos el valor de `tcp-bakclog`, debemos acordarnos de modificar también  `net.core.somaxconn` y 
`tcp_max_syn_backlog`

^^^^^^

### Clientes: `maxclients`

Número máximo de clientes que se pueden conectar simultáneamente.


^^^^^^

#### Clientes: `maxclients`

Este número debe ser inferior al número máximo de ficheros abiertos (`ulimit -n`) menos
32 (descriptores de ficheros que Redis se guarda para uso interno)


^^^^^^

#### Clientes: `maxclients`

Cuando se alcanza el límite de clientes, Redis responde con el siguiente error:

```bash
max number of clients reached
```

^^^^^^

### Clientes: `tcp-keepalive`

Si este valor es distinto de cero, el servidor de Redis envía al cliente paquetes TCP de tipo
`ACK` en el intervalo de tiempo definido (en segundos).

^^^^^^

#### Clientes: `tcp-keepalive`

Si el cliente no responde, se considera que la conexión está muerta y el servidor de Redis la cierra.

^^^^^^

#### Clientes: `tcp-keepalive`

Este parámetro es muy útil cuando entre el cliente y el servidor tenemos cortafuegos o cualquier otro hardware
que pueden cerrar las conexiones sin avisar.

Si un firewall corta la conexión, el cliente se entera cuando intenta hablar con el servidor
y se reconecta.


^^^^^^

#### Clientes: `tcp-keepalive`

Sin embargo, el servidor de Redis no se entera y mantendría la conexión abierta de manera indefinida.

Este parámetro es útil para controlar que el número de conexiones no se dispare en este tipo de escenarios.


^^^^^^

### Clientes: `client-output-buffer-limit`

Antes de enviar la respuesta al cliente, Redis llena un buffer para cada conexión y envía la respuesta
de una sola vez.

```bash 
client-output-buffer-limit <class> <hard limit> <soft limit> <soft seconds>
``` 

Existen varias clases de buffer (`normal`, `replica`, `pubsub`)

^^^^^^

### Clientes: Resumen

| **Parámetro** | **Valor**  |
| --------------: | :---------- |
| `timeout` | 0 |
| `tcp-backlog` | 511 |
| `tmaxclients` | 10.000 | 
| `tcp-keepalive` | 300 |
| `client-output-buffer-limit normal` | 0 0 0 |
| `client-output-buffer-limit slave` | 512mb 256mb 60 |
| `client-output-buffer-limit pubsub` | 32mb 8mb 60 |

^^^^^^

[Más información](https://redis.io/topics/clients)
