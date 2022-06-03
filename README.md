# db2-oefeningen

Verzameling van opgeloste DB2 oefeningen uit projektwerk.

**Zie je een fout? Pull request!**

Neem op tijd pauze!

![MicroPauze](Images/micropauze.png)

## Quickstart using Docker

Als je [Docker](https://www.docker.com/) hebt geinstalleerd, kan je makkelijk lokaal een postgresql container starten. De `docker-compose.yaml` start zo een container op. [De Alpine distro](https://www.alpinelinux.org/) zorgt ervoor dat de container maar 200mb inpakt. Het ander voordeel is dat je alles 'from scratch' kan doen als je een labo opnieuw wil maken.

```console
$ docker-compose up
db_1  |  Starting PostgreSQL ...
```

Je kan via PGAdmin connecteren met de lokale database op `localhost:5432` met user `root`, password `verygood` en database `main`.

Om de container te stoppen, do control+C of `docker-compose down`.

Dankzij de volume mount verlies je geen data on shutdown.

### Zonder docker-compose

> Dit is met binds, dus je container kan aan je host files! Wees zeker dat je de bind aanpast naar een bestaande directory.

```console
$ docker run -d \
  --name postgres \
  -p 5432:5432 \
  --mount type=bind,source=/path/to/data,target=/data/postgres \
  postgres:alpine
```

Om de stoppen:

```console
$ docker stop postgres
```

### Heads up

Je moet voor de eerste keer zelf de schemas aanmaken en seeden. Dit is op de exact dezelfde manier als uitgelegd tijdens de les.
