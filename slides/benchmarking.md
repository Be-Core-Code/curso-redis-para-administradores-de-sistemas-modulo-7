### Benchmarking

Redis nos facilita una herramienta (`redis-benchmark`) para hacer `benchmarking` de nuestro servidor Redis.

^^^^^^

#### Benchmarking

El test más sencillo que podemos ejecutar es ejecutar `redis-benchmark` dentro de la propia máquina virtual:

```bash
> redis-benchmark
====== PING_INLINE ======
  100000 requests completed in 13.71 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1
(...y muchos resultados más)
```

^^^^^^

#### Benchmarking

Si queremos resultados que tengan en cuenta nuestra arquitectura de red, deberemos ejecutar
`redis-benchmark` desde la máquina donde está nuestra aplicación.

^^^^^^

#### Benchmarking

Podemos probar un comando concreto ([`SET`](https://redis.io/commands/set)) con 100 clientes
en paralelo:

```bash
> redis-benchmark -t SET -c 100 -n 10000000 -r 10000000 -d 256 -P 10000
====== SET ======
  10000000 requests completed in 17.99 seconds
  100 parallel clients
  256 bytes payload
  keep alive: 1

0.00% <= 1 milliseconds
0.20% <= 21 milliseconds
...
99.90% <= 7225 milliseconds
100.00% <= 7225 milliseconds
555988.00 requests per second
```

^^^^^^

#### Benchmarking

[Más información - **Lectura muy recomendable / Obligatoria**](https://redis.io/topics/benchmarks)