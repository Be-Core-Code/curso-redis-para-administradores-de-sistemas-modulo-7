### Securizando Redis

**Redis es un servicio que está pensado para ejecutarse en redes de confianza.**

Por defecto, y para evitar despistes, Redis sólo escucha en la interfaz 127.0.0.1.

^^^^^^

#### Securizando Redis

**Sólo permitir el acceso a Redis desde clientes y redes autorizados**.

Por ejemplo, si redis se usa como cache de una aplicación web, únicamente la aplicación
web debería poder conectarse a Redis.

^^^^^^

### Securizando Redis: Unix sockets 

En determinadas circunstancias, puede resultar útil desactivar totalmente el acceso IP a Redis
y obligar a los clientes a que se conecten usando un socker Unix.

```bash
# /etc/redis.conf

port 0
unixsocket /var/run/redis/redis.sock 
unixsocketperm 766
``` 

^^^^^^

#### Securizando Redis: Unix sockets 

Para conectarnos usamos la opción `-s`:

```bash
> redis-cli -s /var/run/redis/redis.sock
```

^^^^^^

### Securizando Redis: Contraseña

Podemos usar la opción `requirepass` para solicitar a los clientes una contraseña antes de 
contectarse a Redis:

```bash
# /etc/redis.conf

requirepass "#eZ$==@0'76uv5j^s#L:,JET-G%d?X4af$EY{`>n4G/=2;,c+^RcQ]`[YCT*:pk"
``` 

^^^^^^

#### Securizando Redis: Contraseña

Si nos conectamos a nuestra instancia e intentamos crear una nueva clave:

```redis-cli
redis-cli > SET miclave 123
(error) NOAUTH Authentication required. 
```

^^^^^^

#### Securizando Redis: Contraseña

Debemos autenticarnos con el comando [`AUTH`](https://redis.io/commands/bgrewriteaof):

```redis-cli
redis-cli > AUTH "#eZ$==@0'76uv5j^s#L:,JET-G%d?X4af$EY{`>n4G/=2;,c+^RcQ]`[YCT*:pk"
OK
redis-cli > SET miclave 123
OK 
```

^^^^^^

#### Securizando Redis: Contraseña

Si estamos utilizando réplicas, tendremos que añadir la opción `masterauth` a las configuración
de la réplica para que se autentique correctamente en el cliente:

```bash
# /etc/redis.conf

masterauth "#eZ$==@0'76uv5j^s#L:,JET-G%d?X4af$EY{`>n4G/=2;,c+^RcQ]`[YCT*:pk"
```

^^^^^^

### Securizando Redis: Ocultando comandos

Redis no implementa ningún sistema de permisos ni de control de acceso a datos.

**Cualquier cliente que se conecte a Redis podrá ejecutar cualquier comando**.

^^^^^^

#### Securizando Redis: Ocultando comandos

Redis facilia una opción de configuración `rename_command` para cambiar el nombre de un comando:

```bash
# /etc/redis.conf
rename_command CONFIG 882ud77yh1298dfa8dfs9 
```

```redis-cli
redis-cli > 882ud77yh1298dfa8dfs9 get save
1) "save"
2) "900 1 300 10 60 10000"
```

^^^^^^

#### Securizando Redis: Ocultando comandos

Para desactivar un comando podemos renombrarlo a una cadena vacía:

```bash
# /etc/redis.conf
rename_command CONFIG "" 
```

^^^^^^

#### Securizando Redis: Ocultando comandos

Renombrar comandos que quedan registrados en el fichero AOF (por ejemplo GET, HMSET, ZADD, etc)
o que se transmiten a las réplicas puede dar problemas y no se recomienda.