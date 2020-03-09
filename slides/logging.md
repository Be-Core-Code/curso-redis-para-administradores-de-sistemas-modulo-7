### Logging

El log de Redis se configura mediante dos parámetros del fichero de configuración:

| *Parámetro*  | |
| --- | --- |
| `loglevel` | `debug`, `verbose`, `notice`, `warning` |
| `logfile` |  "" (empty string) |


^^^^^^

#### Logging

Cuando el `logfile` es una cadena vacía, el log se redirige a la salida estándar.


^^^^^^

#### Logging

**⚠️ Si se utiliza la salida estándar junto con el modo `demonize`, el log se redireccionará a
`/dev/null`**.

^^^^^^

#### Logging

El formato de las líneas de log de redis es:

```bash
pid:role timestamp loglevel message
```

* `pid`: ID del proceso

^^^^^^

#### Logging

* `role`
  * `M`: Master
  * `S`: Slave
  * `C`: RDB/AOF child
  * `X`: Sentinel
  
^^^^^^

#### Logging

* `loglevel`
  * `.`: debug
  * `-`: verbose
  * `*`: notice
  * `#`: warning
  
^^^^^^

### Logging: syslog

Redis permite la integración con syslog a través de los siguientes parámetros de configuración:


| *Parámetro*  | |
| --- | --- |
| `syslog-enabled` | `yes`, `no` |
| `syslog-ident` |  redis |
| `syslog-facility` | local0 |


