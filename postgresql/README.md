# Comando psql para probar conexiones no ssl

```sh
psql "host=acid-ana-cluster port=5432 dbname=postgres user=postgres sslmode=disable"
```

# Configurar el template de cluster para conexiones no SSL

Hay que configurar el parámetro **pg_hba** de la siguiente manera:

```yaml
    - host      all             all          0.0.0.0/0          md5
    - local     all             all                             trust
    - hostssl   all             +zalandos    127.0.0.1/32       pam
    - host      all             all          127.0.0.1/32       md5
    - hostssl   all             +zalandos    ::1/128            pam
    - host      all             all          ::1/128            md5
    - local     replication     standby                         trust
    - hostssl   replication     standby      all                md5
    - host      replication     standby      all                trust
    - hostnossl all             all          all                reject
    - hostssl   all             +zalandos    all                pam
    - hostssl   all             all          all                md5
```

Atención a la primera línea. Es la clave para permitir conexiones *no SSL*. **DEBE IR LA PRIMERA**