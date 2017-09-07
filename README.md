# Info
A docker/symfony docker using [This PHP docker boilerplate](https://github.com/webdevops/php-docker-boilerplate) and [This PHP docker boilerplate](https://github.com/maxpou/docker-symfony)

# Setup
## 1. Clone boilerplate
Cloning include specific configuration
```bash
cd /var/www/project-name
git clone --recursive https://github.com/webdevops/php-docker-boilerplate.git docker
cp conf/docker/bin/create-project.sh docker/bin/.
```

## 2. Override configuration
### Develop
```bash
cp conf/docker/develop/docker-compose.develop.yml docker/docker-compose.yml
cp conf/docker/develop/Dockerfile.development docker/.
cp conf/docker/develop/etc/environment.development.yml docker/etc/.
cp conf/docker/develop/etc/php/development.ini docker/etc/php/.
```

### Production
```bash
cp conf/docker/production/docker-compose.production.yml docker/docker-compose.yml
cp conf/docker/production/Dockerfile.production docker/.
cp conf/docker/production/etc/environment.production.yml docker/etc/.
cp conf/docker/production/etc/php/production.ini docker/etc/php/.
```

## 3. Setup backup folder
```bash
mv docker/backup backup
ln -s ../backup docker/backup
```

## 4. Setup Symfony
```bash
mv docker/app symfony
ln -s ../symfony docker/app
make --directory docker create symfony lts
rm -rf symfony.*
```
### Develop
make sure it match the environment.development.yml
```bash
cp conf/symfony/develop/app/config/parameters.yml symfony/app/config/.
```

### Production
make sure it match the environment.production.yml
```bash
cp conf/symfony/production/app/config/parameters.yml symfony/app/config/.
```

## 5. Run docker
```bash
cd docker
# consider using "make rebuild"
docker-compose up -d
```

## 6. Add docker's domain to host file
```bash
sudo echo $(docker network inspect bridge | grep Gateway | grep -o -E '[0-9\.]+') "symfony.dev" >> /etc/hosts
```


# Load DB structure
```bash
docker exec -it docker_app_1 bash
bin/console doctrine:database:create
bin/console doctrine:schema:update --force
bin/console doctrine:fixtures:load --no-interaction
```

# Useful commands
__Need to run inside /var/www/project-name/docker/__
```bash
# Enter /var/www/project-name/docker/
$ cd /var/www/project-name/docker/
 
# bash commands
$ docker-compose exec app bash
 
# Composer (e.g. composer update)
docker-compose exec app composer update
 
# SF commands (Tips: there is an alias inside php container)
docker-compose exec app bin/console cache:clear 
 
# Retrieve an IP Address (here for the nginx container)
docker inspect --format '{{ .NetworkSettings.Networks.dockersymfony_default.IPAddress }}' $(docker ps -f name=nginx -q)
docker inspect $(docker ps -f name=nginx -q) | grep IPAddress
 
# MySQL commands
docker-compose exec mysql mysql -uroot -p"symfony"
 
# F***ing cache/logs folder
sudo chmod -R 777 var/cache var/logs var/sessions
 
# Check CPU consumption
docker stats $(docker inspect -f "{{ .Name }}" $(docker ps -q))
 
# Delete all containers
docker rm $(docker ps -aq)
 
# Delete all images
docker rmi $(docker images -q)
```

# TODO
! Bug - Why running "docker-compose exec app php /app/bin/console cache:clear" defect Cache/dev by setting the owen to root, instead of the host's username