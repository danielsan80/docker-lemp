Ambiente LEMP per applicazioni PHP
==================================

## Clone
```
git clone git@github.com:matiux/DockerLemp.git
git submodule update --init --recursive
```

## Entrare nei container
* Per entrare nel container php: `./dc exec --user utente php bash`. Equivale a  `docker exec --it --user <GID/USER> <CONTAINER_ID>`

## Utilizzo
Lo script `dc` è una scorciatoia a `docker-compose`
* `./dc` esegue `docker-compose ps`
* `./dc up -d` eseguite il build e l'up dei containers mettendoli poi in backgroud
* `./dc down` stoppa i containers.
* `./dc down -v` per rimuovere i `named volumes` dichiarati nel progetto.
* `./dc --rmi type` per rimuovere le immagini. Type = `all` rimuove tutte le immagini usate da ogni servizio. Type = `local` rimuove solo le immagini che non hanno un tag personalizzato impostato dal campo `image`.
* Collegarsi a mysql/mariadb con il client mysql `mycli`: `mycli -uroot -ppwd -h localhost -P3307`
* Collegarsi a mysql/mariadb con il client mysql classico `mysql`: `mysql -uroot -ppwd -h127.0.0.1 -P3307` (di default il client mysql usa Unix sockets invece di TCP. Quindi bisogna specificare l'IP per esteso e non usare "localhost" perchè lo traduce in unix socket invece di tcp/ip). In alternativa si può fare: `mysql -uroot -p -P3306 -h ip_container`

## Gestione Registry per le proprie immagini
* Creare il repo su docker hub (https://hub.docker.com/)
* Taggare durante il build: `docker build -t matiux/php:fpm-7.1-2.0.0 .`
* Taggare dopo il build: `docker tag <id_immagine> matiux/php:fpm-7.1-2.0.0`
* E' anche possibile aggiungere più tag contemporaneamente: `docker build -t matiux/php:fpm-7.1-2.0.0 -t latest .`
* Se un'immagine ha più tag è possibile rimuovere un tag: `docker rmi matiux/php-fpm-7.1:v2.0.0`. In questo modo non verrà cancellata l'immagine ma solo il tag
* Rinominare un tag: `docker tag <id_immagine> <nuovo_tag>`. Es. `docker tag b7bb3ad94649 matiux/php:7.1-fpm-2.0.0`
* Push: `docker push matiux/php:fpm-7.1-2.0.0`
* Pullare l'immagine sul repo: `docker pull matiux/php:fpm-7.1-2.0.0`

## Comandi utili
* `docker volume ls` per listare i volumi
* `docker volume prune` rimuove tutti i volumi inutilizzati
* `docker volume inspect <id_volume>` informazioni sul volume
* Pulizia totale: `docker system prune -a`
* Ottenere l'ip di un container: `docker inspect --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' nomecontainer`
* Purge all: `docker rm $(docker ps -a -q) && docker rmi $(docker images -q)`
* Stop di tutti i containers: `docker stop $(docker ps -a -q)`
* Rebuild the image to suck in latest `start-container` file: `./dc build`
* Turn on containers, let it use the default "local" environment - xdebug is enabled!:
   ```
   ./dc up -d
   ./dc exec app php --info | grep remote_enable
   ```
* Check that xdebug is not present/enabled when APP_ENV is set to "production":
   ```
   APP_ENV=production ./develop up -d
   ./dc exec app php --info | grep remote_enable
   ```

## Link utili:
* https://docs.docker.com/compose/compose-file/#service-configuration-reference
* https://docs.docker.com/engine/reference/builder/#using-arg-variables
* https://git-scm.com/book/en/v2/Git-Tools-Submodules

## Redis

* Dato che l'immagine è alpine, non c'è bash. Usare quindi: `docker exec -it redis sh`

#### Come usarlo nel codice:

```
$client = RedisAdapter::createConnection(
   'redis://redis_db:6379'
);
```