# Postgres
```yml
# Use postgres/example user/password credentials
version: '3.1'

services:

    postgres:
        image: postgres
        container_name: postgres
        ports:
          - "35432:5432"
        environment:
            POSTGRES_USER: root
            POSTGRES_PASSWORD: P@ssw0rd
```
