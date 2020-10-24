Simple container that I use to run small php projects.

To generate the image, run:
```
docker-compose up --build
```

Once it's generated, to get the container up and running you only need to do:
```
docker-compose up
```

To log into the running container, run:
```
docker exec -it sandbox-fpm bash
```
