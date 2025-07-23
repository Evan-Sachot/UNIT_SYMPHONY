# UNIT_SYMPHONY<img width="960" height="504" alt="image" src="https://github.com/user-attachments/assets/887eca19-b8a9-426b-b50f-6741e022089e" />
<img width="960" height="504" alt="image" src="https://github.com/user-attachments/assets/56f65c1f-5a7a-4b7e-a36d-b3a20572b975" />
<img width="867" height="464" alt="image" src="https://github.com/user-attachments/assets/81843ce7-b271-47c6-a1ab-99daa7d30641" />
<img width="867" height="464" alt="image" src="https://github.com/user-attachments/assets/f12a6459-64e9-46f6-891b-d5f9635ec7df" />
services:  #passage a la partie webserver services
app:
image: php:8.2-fpm      #prise de l'image docker
container_name: symfony_app #nom du conteneur 
working_dir: /var/www/html #montre le dossier de travail
volumes:
- ./app:/var/www/html # Code source persistant
- app_logs:/var/log/php # Logs PHP
- app_cache:/var/www/html/var # Cache et sessions Symfony
networks:
- symfony_network #connexion au reseau partager 

webserver:
image: nginx:stable   #utilisation d'une image Nginx
container_name: symfony_webserver #nom du conteneur
ports:
- "8080:80" #accés au site sur port local//8080
volumes:
- ./app:/var/www/html # Code source
- ./nginx:/etc/nginx/conf.d # Config Nginx
- nginx_logs:/var/log/nginx # Logs Nginx persistants
depends_on:
- app #precice les dependance du code 
networks:
- symfony_network #connexion au reseau partager 

database:     #passage a la partie database

_16

image: mysql:8.0   #image de la database mysql
container_name: symfony_db #nom du conteneur
environment:       #definition des variable d'environnement 
PMA_HOST: symfony_db  #host en symfony 
MYSQL_ROOT_PASSWORD: root #mot de passe du root 
MYSQL_DATABASE: symfony #database en symfony
MYSQL_USER: symfony #utilisateur de la base de donnés 
MYSQL_PASSWORD: symfony #mot de passe de la base de donnés 
ports: #permet de visualiser la database (emplacement)
- "3306:3306"
volumes:
- db_data:/var/lib/mysql # ← Stocke les bases de données et relations internes
networks:
- symfony_network #relie le conteneur au reseau 

adminer:
image: adminer #utilisation de adminer pour interagir avec mysql
container_name: symfony_adminer #nom du conteneur 
restart: always #demarage automatique 
ports:
- "8081:8080" # Adminer sera accessible sur http://localhost:8081
depends_on: #les dependance 
- database
networks: #relie le conteneur au reseau 
- symfony_network

phpmyadmin:  #autre interface pour la base de donnés 
image: phpmyadmin/phpmyadmin #utilise un img de phpadmin
container_name: symfony_phpmyadmin #nom du conteneur 
restart: always #redemarage automatique
ports:
- "8082:80" # phpMyAdmin sera accessible sur http://localhost:8082
environment:
PMA_HOST: symfony_db  #precise qu'il est en symfony

_17

MYSQL_ROOT_PASSWORD: root
PMA_PMADB: phpmyadmin # Nom de la base de configuration
PMA_CONTROLUSER: symfony # Utilisateur qui gère les tables internes
PMA_CONTROLPASS: symfony # Mot de passe du user de contrôle
depends_on:
- database #depend  de la base de donnés 
networks:
- symfony_network #connecte au reseau 
volumes:
- phpmyadmin_data:/var/lib/phpmyadmin # Volume pour les données internes

networks:
symfony_network:
driver: bridge 

volumes:
db_data: # Pour MySQL
phpmyadmin_data: # Pour phpMyAdmin
app_logs: # Logs PHP-FPM
app_cache: # Cache, sessions Symfony
nginx_logs: # Logs Nginx persistants
