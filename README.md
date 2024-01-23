
# Configuração de Infraestrutura AWS

## Criação da VPC:

- VPC e muito mais
- GATEWAYS NAT (USD) - 1 por AZ

## Criação de Security Groups:
- Para as intâncias
- Para o EFS
- Para ELB
- Para RDS

## Criação do EFS:

- Personalizado -> Acesso à rede -> ID-da-sub-rede: Selecione as subnetes públicas da VPC criada anteriormente

## Criação do RDS:

- Criar banco de dados
- Criação padrão
- MySQL nível gratuito
- Nome do usuário principal (*guardar este user*)
- Senha principal (*memorizar a senha pois será usada posteriormente*)
- Em conectividade -> selecione a VPC criada
- Acesso ao público: SIM
- Grupos de segurança da VPC: selecione o grupo criado para o RDS
- Em configuração adicional -> dê um nome para o banco de dados (*memorize*)

## Criação da Instância EC2:

- Tipo -> t3.micro

## Instalação do Docker via user_data.sh:

```bash
#cloud-config
package_update: true
package_upgrade: true
runcmd:
- yum install -y amazon-efs-utils
- apt-get -y install amazon-efs-utils
- yum install -y nfs-utils
- apt-get -y install nfs-common
- file_system_id_1=fs-06fcb65afe79b862d
- efs_mount_point_1=/mnt/efs/fs1
- mkdir -p "${efs_mount_point_1}"
- test -f "/sbin/mount.efs" && printf "\n${file_system_id_1}:/ ${efs_mount_point_1} efs tls,_netdev\n" >> /etc/fstab || printf "\n${file_system_id_1}.efs.us-east-1.amazonaws.com:/ ${efs_mount_point_1} nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev 0 0\n" >> /etc/fstab
- test -f "/sbin/mount.efs" && grep -ozP 'client-info]\nsource' '/etc/amazon/efs/efs-utils.conf'; if [[ $? == 1 ]]; then printf "\n[client-info]\nsource=liw\n" >> /etc/amazon/efs/efs-utils.conf; fi;
- retryCnt=15; waitTime=30; while true; do mount -a -t efs,nfs4 defaults; if [ $? = 0 ] || [ $retryCnt -lt 1 ]; then echo File system mounted successfully; break; fi; echo File system not available, retrying to mount.; ((retryCnt--)); sleep $waitTime; done;

#!/bin/bash
yum update -y
yum install -y docker
service docker start
usermod -a -G docker ec2-user
systemctl enable docker

curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```
## Criação do Load Balancer:
- Nome do Load Balancer
- Esquema -> voltado para a internet
- Tipo de endereço IP -> IPv4
- Mapeamento de rede: VPC -> criada anteriormente
- Mapeamentos -> subnet pública 1a e 1b
- Grupo de segurança: Selecione o grupo criado anteriormente
- Listeners: HTTP:80

## Criação do Target Groups:
- Tipo de destino -> Instâncias
- Protocolo -> IPv4
- VPC -> criada anteriormente
- Registrar destinos -> selecione a instância criada
- Criar grupo de destino

##  Criar o arquivo docker-compose na pasta EFS
- Criação do arquivo docker-compose.yml:
- sudo nano docker-compose.yml

```
version: '3.1'
services:
  wordpress:
    image: wordpress
    volumes:
      - /home/ec2-user/efs/wordpress4:/var/www/html
    ports:
      - 80:80
    restart: always
    environment:
      WORDPRESS_DB_HOST: database-1.************.us-east-1.rds.amazonaws.com                 
      WORDPRESS_DB_USER: admin
      WORDPRESS_DB_PASSWORD: ************
      WORDPRESS_DB_NAME: wordpress_database
```

## Criação de uma AMI:
- Instâncias -> (botão direito) Imagem e modelos -> Criar imagem
- Nome: Criar Imagem

## Criar Auto Scaling
- Configuração do Auto Scaling:
- Criar grupo do Auto Scaling
- Criar modelo de execução: nome, inserir tags
- Minhas AMIs -> selecione a AMI criada
- Criar modelo de execução
- De volta ao auto scaling: selecionar modelo de execução
- Anexar um balanceador de carga existente -> selecione o criado anteriormente
- Capacidade máxima: 2
- Próximo
- Próximo
- Criar Grupo do Auto Scaling
