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
- [Restante do script para montagem do EFS e instalação do Docker]

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
- Criação do arquivo docker-compose.yml:

"
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
"
#
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
