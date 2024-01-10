# atividae2
Atividade 2 Compass

Ao criar a instância:
Detalhes avançados:
	#!/bin/bash

	# Instalar Docker
	sudo yum install docker -y

	# Ativar e iniciar o serviço do Docker
	sudo systemctl enable docker.service
	sudo systemctl start docker.service

-----------------------------------------------
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo nano docker-compose.yml:

"
version: '3.7'
services:
  wordpress:
   image: wordpress
   volumes:
     - /efs/website:/var/www/html
   ports:
     - "80:80"
   restart: always
   envioronment: 
     WORDPRESS_DB_HOST: wordpress.xxxxxxxxx.us-east-1.rds.amazonaws.com
      WORDPRESS_DB_USER: adminWordPress
      WORDPRESS_DB_PASSWORD: xxxxxxxx
      WORDPRESS_DB_NAME: wordpressdatabase
"
---------------------------------------------------------
