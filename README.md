# TD 1 : Vendre des livres numériques avec WooCommerce

## 2. Démarrer un wordpress via docker compose

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
        image: mysql:5.7
        volumes:
            - ./mysql/database:/var/lib/mysql
        restart: always
        environment:
            MYSQL_ROOT_PASSWORD: mypassword
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wordpress
            MYSQL_PASSWORD: wordpress

    wordpress:
        image: wordpress:latest
        depends_on:
            - db
        ports:
            - 80:80
        restart: always
        environment:
            WORDPRESS_DB_HOST: db:3306
            WORDPRESS_DB_USER: wordpress
            WORDPRESS_DB_PASSWORD: wordpress
        volumes:
            - ./wordpress/aixmazone.ini:/usr/local/etc/php/conf.d/aixmazone.ini:ro
            - ./wordpress/src:/var/www/html
```

### Démarrer les services avec docker compose

```shell
docker-compose up -d
```

## 3. Installer l'extension Woocommerce

http://localhost:8080/wp-admin
Extension ⇒ Ajouter ⇒ chercher “woocommerce” ⇒ Installer.

