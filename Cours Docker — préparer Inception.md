# Cours Docker — préparer Inception

Oct 22, 2026 · @Arnaud

## Vue d'ensemble

Inception te fait monter, dans une machine virtuelle, un site WordPress réparti sur trois conteneurs (NGINX, WordPress avec php-fpm, MariaDB), reliés par un réseau Docker et construits par un Makefile et Docker Compose.

Le navigateur parle en HTTPS au port 443 de NGINX, seule porte d'entrée de l'infrastructure. NGINX sert les fichiers statiques et transmet les pages PHP à php-fpm sur le port 9000. WordPress lit et écrit dans MariaDB sur le port 3306. Deux volumes gardent les données quand les conteneurs sont détruits : la base de données et les fichiers du site.

Ce cours suit l'ordre de tes notes : le vocabulaire, les quatre comparaisons demandées (machine virtuelle vs Docker, secrets vs variables d'environnement, réseau Docker vs réseau host, volumes vs bind mounts), la construction des services, puis la préparation de la soutenance.

| Dans tes notes | Où le trouver |
| --- | --- |
| lexical : WordPress, Docker Compose, image, conteneur, Dockerfile, volume, TLS, Redis, FTP | Lexique |
| Virtual Machines vs Docker | Docker et machine virtuelle |
| Docker image, Dockerfiles, Alpine / Debian, pas DockerHub, tag latest | Images, conteneurs et Dockerfile |
| daemons, PID 1, hacky patches | PID 1, démons et « hacky patches » |
| docker-compose.yml, restart, ligne network | Docker Compose |
| Docker Network vs Host Network, network: host, links | Réseau |
| Docker Volumes vs Bind Mounts, /home/login/data | Volumes |
| Secrets vs Environment Variables, .env, mots de passe dans Git | Secrets et variables d'environnement |
| NGINX en TLS, WordPress + php-fpm, MariaDB, utilisateurs WordPress | Les trois services |
| login.42.fr | Nom de domaine et machine virtuelle |
| Makefile | Makefile |
| redis cache, FTP server container | Bonus |
| ls -alR, cat srcs/.env | Commandes et vérifications |

### Structure du dépôt

Le sujet impose un dossier `srcs/` pour toute la configuration et un `Makefile` à la racine. Le reste de l'arborescence ci-dessous est l'organisation habituelle de ce projet ; tu peux l'afficher avec `ls -alR`.

```text
.
├── Makefile
├── USER_DOC.md
├── DEV_DOC.md
├── secrets/              (hors Git)
│   ├── db_password.txt
│   ├── db_root_password.txt
│   └── wp_admin_password.txt
└── srcs/
    ├── docker-compose.yml
    ├── .env
    └── requirements/
        ├── nginx/        (Dockerfile, conf/)
        ├── wordpress/    (Dockerfile, conf/, tools/)
        └── mariadb/      (Dockerfile, conf/, tools/)
```

## Lexique

Chaque terme de ta liste « lexical » est défini ici en une ou deux phrases ; les chapitres suivants les développent.

| Terme | Définition |
| --- | --- |
| WordPress | Logiciel de gestion de contenu écrit en PHP. Le code génère les pages, la base de données garde les articles et les réglages, le disque garde les images et les extensions. |
| NGINX | Serveur web et reverse proxy. Dans le projet, il est la seule entrée (port 443, HTTPS) et transmet les pages PHP à php-fpm. |
| php-fpm | Gestionnaire de processus PHP qui parle le protocole FastCGI. NGINX ne sait pas exécuter PHP : il envoie la requête à php-fpm, qui lance le script et renvoie la page. |
| MariaDB | Serveur de base de données relationnelle, issu de MySQL. Il stocke la base de WordPress. |
| Image | Modèle en lecture seule, fait de couches, qui contient un système de fichiers minimal et le programme à lancer. Elle s'obtient avec `docker build` (ou `docker compose build`) à partir d'un Dockerfile. |
| Conteneur | Instance en cours d'exécution d'une image : des processus isolés (processus, réseau, fichiers) qui partagent le noyau de la machine. Supprimé, il perd ses données internes, d'où les volumes. |
| Dockerfile | Fichier texte qui décrit, instruction par instruction, comment construire une image. Le sujet en demande un par service, écrit par toi. |
| Docker Hub, Alpine, Debian | Docker Hub est le registre public d'images. Tu n'y prends que l'image de base Alpine ou Debian ; les images nginx, wordpress et mariadb sont construites par toi. |
| Docker Compose | Outil qui décrit plusieurs conteneurs dans un fichier YAML (`docker-compose.yml`) : images à construire, réseau, volumes, variables, redémarrage. Il se lance avec `docker compose`. |
| Volume | Stockage géré par Docker qui survit à la suppression d'un conteneur. Le projet en demande deux : la base de données et les fichiers du site. |
| Réseau Docker | Réseau virtuel privé où les conteneurs se joignent par le nom de leur service. |
| Démon (daemon) | Programme qui se détache du terminal pour tourner en arrière-plan. Dans un conteneur, un service qui se détache laisse le processus principal sans travail : il se termine et le conteneur s'arrête avec lui. |
| PID 1 | Premier processus d'un conteneur. Il reçoit les signaux d'arrêt, récupère les processus orphelins, et le conteneur s'arrête quand il se termine. |
| Hacky patches | Les bricolages interdits qui gardent un conteneur en vie sans vrai service : `tail -f`, `sleep infinity`, `while true`, `bash`. |
| TLS 1.2 / 1.3 | Protocole qui chiffre et authentifie la connexion HTTPS. Le sujet n'autorise que ces deux versions ; SSL, TLS 1.0 et 1.1 sont dépréciés. |
| Redis | Base de données en mémoire, très rapide. En bonus, WordPress s'en sert comme cache d'objets pour soulager MariaDB. |
| Serveur FTP | Service de transfert de fichiers. En bonus, un conteneur FTP pointe vers le volume des fichiers WordPress. |
