# TD 1 : Vendre des livres numériques avec WooCommerce

## 1. Démarrer un wordpress via docker compose sur Github CodeSpace

<details>
<summary>1. Augmenter la taille des fichiers qu'on peut uploader dans wordpress</summary>

Créer le fichier `wordpress/aixmazone.ini`:
```ini
upload_max_filesize = 128M
post_max_size = 128M
max_execution_time = 300
max_input_time=300
```
</details>

<details>
<summary>2. Créer un fichier docker compose</summary>

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
            - ./wordpress/codebase-dynamic-hostname.php:/var/www/html/codebase-dynamic-hostname.php:ro
            - ./wordpress/src:/var/www/html
```
</details>
<details>
<summary>3. Créer le fichier wordpress/codebase-dynamic-hostname.php</summary>

```shell
cat > wordpress/codebase-dynamic-hostname.php <<EOF
<?php
define( 'WP_HOME', 'https://$(jq -r ".CODESPACE_NAME" /workspaces/.codespaces/shared/environment-variables.json)-80.preview.app.github.dev');
define( 'WP_SITEURL', 'https://$(jq -r ".CODESPACE_NAME" /workspaces/.codespaces/shared/environment-variables.json)-80.preview.app.github.dev');
EOF
```
</details>
<details>
<summary>4. Récupérer le host créé dynamiquement dans codespace pour l'utiliser dans la config wordpress</summary>

- `docker compose up` puis CTRL+C pour recopier les fichiers wordpress localement dans le volume monté et tuer les services
- Éditer le fichier `wordpress/src/wp-config.php`, ajouter sur la 2ème ligne: `require_once __DIR__ . '/codebase-dynamic-hostname.php';`
```

</details>
<details>
<summary>4. Démarrer les services avec docker compose</summary>

```shell
docker-compose up -d
```
</details>

## 2. Installer l'extension Woocommerce

Ouvrir wordpress sur /wp-admin
Extension ⇒ Ajouter ⇒ chercher “woocommerce” ⇒ Installer.

