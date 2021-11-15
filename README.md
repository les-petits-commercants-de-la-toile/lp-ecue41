# TD 1 : Vendre des livres numériques avec WooCommerce

## 1. S'assurer d'avoir docker compose

```shell
echo 'export PATH=~/bin:$PATH' >> ~/.bashrc
mkdir -p ~/bin
cd ~/bin
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o ~/bin/docker-compose
chmod +x ~/bin/docker-compose
source ~/.bashrc
```

## 2. Démarrer un wordpress via docker compose

### Préparer l'environnement docker

```shell
# variables d'environnement pour contourner des limitations docker
grep -c 'export USER_ID' &>/dev/null && echo "export USER_ID=$(id -u)" >> ~/.bashrc
grep -c 'export GROUP_ID' &>/dev/null && echo "export GROUP_ID=$(id -g)" >> ~/.bashrc
source ~/.bashrc
```

```shell
# répertoires pour persister sur l'hôte local
mkdir -p td1
chmod a+rwx ~/
chmod a+rwx td1
cd td1
mkdir -p wordpress/src mysql/database
chmod a+rwx -R wordpress mysql
```

Créer le fichier `wordpress/Dockerfile`:
```dockerfile
FROM wordpress:latest
ARG USER_ID
ARG GROUP_ID
RUN sed -i -e "s/33:33/${USER_ID}:${GROUP_ID}/g" /etc/passwd \
  && sed -i -e "s/33:/${GROUP_ID}:/g" /etc/group \
  && chown -R www-data:www-data /var/www
```

Créer le fichier `mysql/Dockerfile`:
```dockerfile
FROM mysql:5.7
ARG USER_ID
ARG GROUP_ID
RUN sed -i -e "s/999:999/${USER_ID}:${GROUP_ID}/g" /etc/passwd \
  && sed -i -e "s/999:/${GROUP_ID}:/g" /etc/group \
  && chown -R mysql:mysql /var/lib/mysql
```

### Augmenter la taille des fichiers qu'on peut uploader dans wordpress


Créer le fichier `wordpress/aixmazone.ini`:
```ini
upload_max_filesize = 128M
post_max_size = 128M
max_execution_time = 300
max_input_time=300
```

### Créer un fichier docker compose

```yaml
version: '3.8'

services:
    db:
        image: aixmazone/mysql
        build:
          context: mysql
          args:
            USER_ID: ${USER_ID}
            GROUP_ID: ${GROUP_ID}
        volumes:
            - ./mysql/database:/var/lib/mysql
        restart: always
        #user: mysql # uncomment on subsequent starts
        environment:
            MYSQL_ROOT_PASSWORD: mypassword
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wordpress
            MYSQL_PASSWORD: wordpress

    wordpress:
        image: aixmazone/wordpress
        build:
          context: wordpress
          args:
            USER_ID: ${USER_ID}
            GROUP_ID: ${GROUP_ID}
        depends_on:
            - db
        ports:
            - 8080:80
        restart: always
        environment:
            WORDPRESS_DB_HOST: db:3306
            WORDPRESS_DB_USER: wordpress
            WORDPRESS_DB_PASSWORD: wordpress
        volumes:
            - ./wordpress/aixmazone.ini:/usr/local/etc/php/conf.d/aixmazone.ini:ro
            - ./wordpress/src:/var/www/html
```

### Copier les fichiers source de docker sur le filesystem local


```shell
docker-compose build
CID=$(docker create --entrypoint=scratch aixmazone/wordpress)
docker cp $CID:/usr/src/wordpress/. wordpress/src/
docker rm $CID
```

### Démarrer les services avec docker compose

```shell
docker-compose up
```

## 3. Installer l'extension Woocommerce

http://localhost:8080/wp-admin
Extension ⇒ Ajouter ⇒ chercher “woocommerce” ⇒ Installer.

