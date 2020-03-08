### Configuración de Linux

Como dijimos en el primer módulo del curso, la forma recomendada de desplegar Redis en producción
es utilizando Linux como sistema operativo

notes:

El motivo por el que no se recomienda Windows es que este sistema operativo no implementa `fork()`.


^^^^^^

### Configuración de Linux: overcommit_memory

Cuando ejecutamos [`BGSAVE`](https://redis.io/commands/bgsave), Redis utiliza un algoritmo
basado en Copy-On-Write (COW). De esta manera no hace falta duplicar los datos en memoria para
guardarlos.

Linux, sin embargo, puede chequear si tiene memoria suficiente para que Redis pueda 
duplicar todas las páginas de memoria, lo que puede dar lugar a que el kernel lo mate (OOM). 

^^^^^^

#### Configuración de Linux: overcommit_memory

Si esto ocurre, veremos en el log los siguientes mensajes:

```bash
> [1524] 24 Sep 10:00:56.037 # Can't save in background: fork: Cannot allocate memory
```

^^^^^^

#### Configuración de Linux: overcommit_memory

Al inicio, Redis verifica si este parámetro está activo:

```bash
4781:M 06 Mar 2020 10:13:49.421 # WARNING overcommit_memory is set to 0! 
Background save may fail under low memory condition. 
To fix this issue add 'vm.overcommit
```

^^^^^^

#### Configuración de Linux: overcommit_memory

```bash
> sudo sysctl -w vm.overcommit_memory=1
> echo vm.overcommit_memory=1" >> /etc/sysctl.conf
```

^^^^^^

#### Configuración de Linux: overcommit_memory

Al configurar este parámetro, cuando se hace una llamada a `malloc()`, esta siempre tendrá éxito,
aunque el sistema no disponga de toda la memoria que se le está pidiendo.

^^^^^^

### Configuración de Linux: swappiness

Este parámetro indica cuán agresivamente el Kernel de Linux paginará memoria al disco.

^^^^^^

#### Configuración de Linux: swappiness

El proceso de paginación tiene una penalización muy alta desde el punto de vista de Redis, ya que
el proceso quedaría bloqueado por las operaciones de disco.

Recordemos que redis se ejecuta en un sólo hilo de forma que mientras está esperando a que termine
el operación de disco **no responde a ninguna otra petición**.

^^^^^^

#### Configuración de Linux: swappiness

Moraleja: **no queremos que se use la memoria de intercambio** 

```bash
> sudo sysctl -w vm.swappiness=0
> echo "vm.swappiness=0" >> /etc/sysctl.conf
```

^^^^^^

### Configuración de Linux: _transparent huge page_

La opción _transparent huge page_ suele estar activa por defecto en Linux.

Cuando está activa, el proceso de `fork` cuando se ejecuta un [`BGSAVE`](https://redis.io/commands/bgsave)
puede ralentizarse por lo que es recomendable desactivar esta opción:

```bash
> echo never > /sys/kernel/mm/transparent_hugepage/enabled
> echo never > /sys/kernel/mm/transparent_hugepage/defrag
```

^^^^^^

#### Configuración de Linux: _transparent huge page_

Redis nos muestra un aviso en el log si no está desactivada esta opción:

```bash
4781:M 06 Mar 2020 10:13:49.421 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. 
This will create latency and memory usage issues with Redis. To fix this issue run the 
command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to 
your /etc/rc.local in order to retain the setting after a reboot. Redis must be 
restarted after THP is disabled.
```

^^^^^^

### Configuración de Linux: net.core.somaxconn 

`net.core.somaxconn` marca el tamaño máximo del backlog de la función `listec()` del socket.

Si es muy corto y tenemos muchos clientes conectándose a Redis, las conexiones se empezarán
a no aceptar (`DROP`)

```bash
> sudo sysctl -w net.core.somaxconn=65535
> echo "net.core.somaxconn=65535" >> /etc/sysctl.conf
```

^^^^^^

#### Configuración de Linux: net.core.somaxconn 

Redis tiene un parámetro, `tcp-backlog`, que tiene un valor de `511` por defecto. 

**Si `net.core.somaxconn` no es mayor que `tcp-backlog`, Redis nos dará un warning al arrancar:**

```bash 
4782:M 08 Mar 2020 20:01:55.283 # WARNING: The TCP backlog setting of 511 cannot be enforced because 
/proc/sys/net/core/somaxconn is set to the lower value of 128.
```

^^^^^^

### Configuración de Linux: tcp_max_syn_backlog 

Este parámetro indica cuál es la longitud máxima de la cola de conexiones que todavía no han 
recibido el ACK del cliente.

```bash
> sudo sysctl -w net.ipv4.tcp_max_syn_backlog=65535
> echo "net.ipv4.tcp_max_syn_backlog=655350" >> /etc/sysctl.conf
```

^^^^^^

### Configuración de Linux: ulimit -n 

`ulimit -n`: Número máximo de ficheros abiertos.
 
El valor por defecto (1024) suele ser demasiado baja. Se recomienda al menos 10.000.

^^^^^^

#### Configuración de Linux: ulimit -n 

En caso de que se intenten abrir más ficheros de los permitidos, veremos el siguiente warning en el
log de Redis:
 
```bash
4911:M 08 Mar 2020 20:09:26.368 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate 
for low ulimit. If you need higher maxclients increase 'ulimit -n'.
```

^^^^^^

#### Configuración de Linux: ulimit -n 

Para fijar este parámetro cuando se vuelva a arrancar el sistema, normalmente
necesitamos editar el fichero `/etc/limits.conf` o `/etc/security/limits.conf`.

```bash
# /etc/security/limits.conf
redis soft nofile 288000
redis hard nofile 288000
```

notes:

En Alpine Linux será necesario instalar el paquete `linux-pam` y `shadow` para disponer de esta configuración.


^^^^^^

### Configuración de Linux: Resumen

| **Parámetro** | **Valor**  |
| --------------: | :---------- |
| `net.core.somaxconn` | 65535 |
| `net.ipv4.tcp_max_syn_backlog` | 65535 |
| `transparent_hugepage/enabled` | never | 
| `transparent_hugepage/defrag` | never |
| `net.core.somaxconn` | 65535 |
| `net.ipv4.tcp_max_syn_backlog` | 65535 |
| `ulimit -n` | 288000 |


^^^^^^

### 💻️ Ejercicio

Incluir esta configuración en el fichero `Vagrantfile` para que se aprovisione automáticamente cuando 
creemos la máquina virtual.  
