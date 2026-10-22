lexical:
- wordpress
- docker compose ?
- Docker image ? build them ?
- container ?	-> Alpine/ Debian not DockerHub
- dockerfiles ? 1 per service ?
- docker-compose.yml
- volume ?
- hacky patches ?
- daemons work ? good/bad ?
-  PID 1 and the best practices for writing Dockerfiles
-  the TLSv1.2 or TLSv1.3 protocol
-  redis cache
-  FTP server container

REQUIREMENTS:  
- Virtual Machines vs Docker
- Secrets vs Environment Variables
- Docker Network vs Host Network
- Docker Volumes vs Bind Mounts


set up:
- Docker container that contains NGINX with TLSv1.2 or TLSv1.3 only.
- Docker container that contains WordPress + php-fpm (it must be installed and
configured) only, without nginx.
- Docker container that contains MariaDB only, without nginx.
- volume that contains your WordPress database.
- second volume that contains your WordPress website files.
- You must use Docker named volumes for these two persistent storages. Bind
mounts are not allowed for these volumes.
- These named volumes must be configured so that their datends up in /home/login/data
on the host machine. They remain Docker named volumes, not bind mounts. Replace "login" with your learner’s username.
- docker-network that establishes the connection between your containers.
Your containers have to restart in case of crash.

-> 



FORBIDDEN !
-  using network: host or --link or links
-  network line must be present in your docker-compose.yml file
-  containers must not be started with a command running an infinite loop
-  prohibited hacky patches: tail -f, bash, sleep infinity, while true
-  can't use latest tag
-  no password in dockerfiles
-  use .env variables
-  Any credentials, API keys, or passwords found in your Git repository (outside of properly configured secrets) will result in project failure

RECOMMENDED
- use Docker secrets to store any confidential information
- Your NGINX container must be the only entrypoint into your infrastructure via the port 443 only, using the TLSv1.2 or TLSv1.3 protocol
- You can store your variables (as a domain name) in an environment variable file like .env
- restart when crash

COMMANDS:
- ```ls -alR```
- ```cat srcs/.env```

Wordpress data base:
- 2 users : 
	-> 1 admin: password can't have  admin/Admin or administrator/Administrator


Volumes:
- available in the /home/login/data folder of the host machine, replace logins
- configure domain  name to point to your local IP address
  -> name: <login>.42.fr


Makefile:
-  Build and launch the project using the Makefile and Docker Compose
-   Use relevant commands to manage the containers and volumes