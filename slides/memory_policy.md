### Memory Policy

Podemos obtener el uso actual de memoria de nuestro servidor Redis usando el comando [`INFO`](https://redis.io/commands/info):

```redis-cli
redis-cli > info memory
      # Memory
      used_memory:462270695
      used_memory_human:440.86M
      used_memory_rss:539832320
      used_memory_rss_human:514.82M
      used_memory_peak:462270695
      used_memory_peak_human:440.86M
      used_memory_peak_perc:100.00%
      used_memory_overhead:97377930
      used_memory_startup:550068
      used_memory_dataset:364892765
      used_memory_dataset_perc:79.03%
      allocator_allocated:462237837
      allocator_active:539794432
      allocator_resident:539794432
      total_system_memory:737271808
      total_system_memory_human:703.12M
      used_memory_lua:37888
      used_memory_lua_human:37.00K
      used_memory_scripts:0
      used_memory_scripts_human:0B
      number_of_cached_scripts:0
      maxmemory:0
      maxmemory_human:0B
      maxmemory_policy:noeviction
      allocator_frag_ratio:1.17
      allocator_frag_bytes:77556595
      allocator_rss_ratio:1.00
      allocator_rss_bytes:0
      rss_overhead_ratio:1.00
      rss_overhead_bytes:37888
      mem_fragmentation_ratio:1.17
      mem_fragmentation_bytes:77594483
      mem_not_counted_for_evict:0
      mem_replication_backlog:0
      mem_clients_slaves:0
      mem_clients_normal:49694
      mem_aof_buffer:0
      mem_allocator:libc
      active_defrag_running:0
      lazyfree_pending_objects:0 
```

^^^^^^

#### Memory Policy

Ahora mismo, estamos usando 462270695 bytes.

A modo de ejemplo, vamos a limitar la memoria a 462271000 bytes.

```redis-cli
redis-cli > CONFIG SET MAXMEMORY 462271000
``` 

^^^^^^

#### Memory Policy

Si intentamos crear una clave, recibiremos el error de que no tenemos memoria suficiente para
crear esa clave:

```redis-cli
redis-cli > HMSET misdatos c1 v1 c2 v2 c3 v3 c4 v4 c5 v5
(error) OOM command not allowed when used memory > 'maxmemory'. 
```

^^^^^^

#### Memory Policy

¿Qué memory policy tenemos activa en este momento?

```redis-cli
redis-cli >config get maxmemory-policy
1) "maxmemory-policy"
2) "noeviction" 
```

^^^^^^

#### Memory Policy

Con esta política se genera un error con cualquier comando que suponga usar más memoria de la permitida.

Si podríamos, por ejemplo, borrar una clave con [`DEL`](https://redis.io/commands/del).

^^^^^^

#### Memory Policy

| maxmemory-policy |  |
| --- | --- |
| `noeviction` | No se borra ninguna clave |
| `allkeys-lru` | Borra claves usando un algoritmo _Least Recently Used (LRU)_ |
| `volatile-lru` | Borra claves con un tiempo de expiración usando un algoritmo _Least Recently Used (LRU)_. Si no encuentra ninguna que borrar, usa `noeviction`|

^^^^^^

#### Memory Policy

| maxmemory-policy |  |
| --- | --- |
| `allkeys-lfu` | Borra claves usando un algoritmo _Least Frequently Used (LFU)_ |
| `volatile-lfu` |Borra claves con un tiempo de expiración usando un algoritmo _Least Frequenlty Used (LRU)_. Si no encuentra ninguna que borrar, usa `noeviction` |

^^^^^^

#### Memory Policy

| maxmemory-policy |  |
| --- | --- |
| `allkeys-random` | Borra una clave aleatoriamente |
| `volatile-random` | Borra claves aleatorias que tienen definido un tiempo de expiración. Si no encuentra ninguna que borrar, usa `noeviction` |

^^^^^^

#### Memory Policy

| maxmemory-policy |  |
| --- | --- |
| `volatile-ttl` | Borra la clave que esté más cerca de expirar. Si no encuentra ninguna que borrar, usa `noeviction` |

^^^^^^

#### Memory Policy

Es muy importante definir una política de gestión de memoria en producción, así como un valor para `maxmemroy`.

notes:

El valor por defecto en muchas distribuciones Linux es `maxmeroy 0`, que no impone ningún límite de memoria a Redis.

^^^^^^

#### Memory Policy

Mencionar que el valor de `used_memory` que nos facilita el comando [`INFO`](https://redis.io/commands/info) incluye:

* Los datos
* Los `client-buffers` que vimos en la sección [Configuración de clientes](/#client_connections_options)

^^^^^^

#### Memory Policy

No es conveniente asignar toda la memoria de la máqina para Redis. Debemos dejar memoria disponible para:

* El correcto funcionamiento del sistema operativo y otros procesoso (monitorización, ssh, etc)
* Los procesos en segundo plano que se lanzan con [`BGSAVE`](https://redis.io/commands/bgsave)
 