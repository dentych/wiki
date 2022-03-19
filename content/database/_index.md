---
title: Database setup
---

Hosted at Hetzner ({{<newtablink login "https://console.hetzner.cloud">}})

Docker-compose for the setup:
```yaml
version: "3.6"

services:
  postgres:
    image: postgres
    restart: always
    volumes:
      - "./postgres/postgres_data:/var/lib/postgresql/data"
    ports:
      - 5432:5432
    environment:
      POSTGRES_PASSWORD: <redacted>
```