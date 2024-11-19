<img width=100% src="https://capsule-render.vercel.app/api?type=waving&color=9B79FC&height=120&section=header"/>
 
# Infraestrutura De Rede

<br>

## 🎯 Objetivo
Projeto acadêmico com o objetivo de desenvolver uma infraestrutura de rede para a empresa fictícia XPTO, que atenda aos seguintes requisitos:
1. Load Balancer: Implementar um sistema de balanceamento de carga utilizando, no mínimo, 3 máquinas para distribuir o tráfego.
2. Proxy Reverso: Configurar uma máquina para atuar como proxy reverso, gerenciando as requisições dos usuários para os servidores apropriados.
3. Banco de Dados: Implementar uma máquina dedicada para o banco de dados, garantindo a segurança e integridade dos dados.
4. VPN: Configurar o acesso à rede através de uma VPN, garantindo que todas as comunicações sejam seguras.
5. Docker: Utilizar o Docker para criar os servidores web e o banco de dados, garantindo portabilidade e fácil gestão dos serviços.

## 📹 Topologia De Rede

![Topologia](./images/topologia.png)

## 🛠️ Tecnologias Utilizadas

* AWS (EC2 e RDS) <img align="center" alt="React" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/amazonwebservices/amazonwebservices-original-wordmark.svg">
* Docker <img align="center" alt="Node" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/docker/docker-original.svg" />
* NGINX <img align="center" alt="Js" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/nginx/nginx-original.svg">
* MongoDB <img align="center" alt="Js" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/mongodb/mongodb-original.svg">
* React <img align="center" alt="Js" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/react/react-original.svg">
* JavaScript <img align="center" alt="Js" height="25" width="35" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/javascript/javascript-plain.svg">
* Ubuntu <img align="center" alt="ubuntu" height="30" width="40" src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/ubuntu/ubuntu-original.svg" >

## 🚀 Começando

Essas instruções permitirão que você obtenha uma cópia do projeto.<br/>

### 1° Passo - Crie o banco de dados MySQL usando RDS
- Criar uma instância RDS com as seguintes configurações.
```
Criação padrão
Tipo do mecanismo - MySQL
Edição do mecanismo - MySQL Commuunity
Versão do mecanismo - MySQL 8.0.39
Modelos - Nível gratuito
Disponibilidade e durabilidade - Instância de banco de dados única
Configurações de credenciais:
    Nome do usuário principal - admin
    Gerenciamento de credenciais - Autogerenciada
    Senha principal - minha_senha
Classe da instância de banco de dados - Classes padrão (db.m7g.large)
Conectividade:
    Não se conectar a um recurso de computação do EC2
    vpc - default
    Acesso público - Sim
```

- Conecte-se ao banco de dados com o endpoint gerado e crie a tabela **to-do-list-db** com o comando.
```
CREATE DATABASE `to-do-list-db`;
```

### 2° Passo - Colocar o projeto em três instância EC2

- Criar três instâncias EC2 com as seguintes configurações.
```
Sistema Operacional - Ubuntu (64 bits)
Tipo de instância - t2.micro
Grupo de segurança - Libere as portas 443 (HTTPS), 80 (HTTP), 22(SSH), 3306 (MySQL), 3000 e 8800
Armazenamento - 1x 30 GiB gp3
```
- Conecte-se a instância da forma que preferir.
- Dentro de cada máquina executar os comandos a seguir para usar o docker (Após esses comandos reinicie a máquina).
```
sudo apt update
sudo apt upgrade
sudo apt install docker.io -y
sudo apt install docker-compose -y
sudo usermod -aG docker $USER (Para usar o docker sem sudo)
sudo apt update
```

- Crie uma pasta chamada app em **/home/ubuntu** e copie o projeto que será utilizado dentro dela.
```
mkdir ~/app
git clone https://github.com/NatanaelSM/to-do-list.git
```

- Dentro do app crie um arquivo chamado **compose.yaml** com as configurações abaixo.
```
services:

  backend:
    container_name: backend
    build:
      context: ./server 
      dockerfile: Dockerfile
    env_file:
      - ./server/.env 
    ports:
      - "8800:8800"

  frontend:
    container_name: frontend
    build:
      context: ./frontend 
      dockerfile: Dockerfile
    ports:
      - "80:80" 
    depends_on:
      - backend 

volumes:
  db_data:
```

- Vá para o diretório **~/app/frontend** e crie um arquivo chamado **Dockerfile** com as configurações abaixo.
```
FROM node:22-alpine AS build

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

FROM nginx:alpine

COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80 

CMD ["nginx", "-g", "daemon off;"]
```

- Vá para o diretório **~/app/server** e crie um arquivo chamado **Dockerfile** com as configurações abaixo.
```
FROM node:22-alpine

WORKDIR /app

COPY ./package*.json ./

RUN npm install

COPY . .

EXPOSE 8800

CMD ["node", "index.js"]
```

- Crie um arquivo **.env** em **~/app/server** com as seguintes configurações (O host será o endpoint gerado no serviço RDS, o endpoint abaixo é somente um exemplo).
```
PORT="8800"
PASSWORD_DB="minha_senha"
USER_DB="admin"
PORT_DB="3306"
HOST_DB="db-mysql.cr0q0rkc6gza.us-east-1.rds.amazonaws.com"
DATABASE_DB="to-do-list-db"
SECRET="minha_secret"
```

- Dentro do diretório **~/app** digite o comando abaixo.
```
docker-compose up --build
```

### 3° Passo - Criar e configurar a instância EC2 que irá fazer o proxy reverso e o load balance com NGINX

- Criar uma instância EC2 com as seguintes configurações.
```
Sistema Operacional - Ubuntu (64 bits)
Tipo de instância - t2.micro
Grupo de segurança - Libere as portas 443 (HTTPS), 80 (HTTP), 22(SSH) e 1194 (UDP)
Armazenamento - 1x 30 GiB gp3
```

- Conecte-se a instância da forma que preferir.
- Dentro da máquina executar os comandos a seguir para configurar o Nginx.
```
sudo apt update

sudo apt upgrade

sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
    | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list

echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
    | sudo tee /etc/apt/sources.list.d/nginx.list

echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
    | sudo tee /etc/apt/preferences.d/99nginx

sudo apt install nginx
```

- Criar um arquivo dentro do diretório **/etc/nginx/conf.d** para definir e configurar os loads balances.
```
cd /etc/nginx/conf.d
mkdir loadbalancer.conf
```
- Editar **loadbalancer.conf** com as configurações a seguir (Substitua os IP's pelos das instâncias EC2 criadas anteriormente).
```
upstream loadbalances {
	server 12.34.56.789 weight=3;
	server 12.34.56.789 weight=2;
	server 12.34.56.789 weight=1;
}
```

- Dentro do arquivo /etc/nginx/conf.d/default.conf inclua o arquivo que contém os load balances e configure-o da seguinte forma.
```
include /etc/nginx/conf.d/webservers.conf;

server {
    listen       80;
    server_name  192.168.0.100;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
	proxy_pass http://webservers;
	proxy_buffering off; 
	proxy_http_version 1.1;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection "upgrade";	proxy_set_header Host $host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header X-Forwarded-Proto $scheme;
	proxy_read_timeout 200;
	proxy_connect_timeout 200;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

- Reinicie o Nginx
```
sudo systemctl restart nginx
```

- Executar os comandos a seguir para configurar o VPN usando o OpenVPN.
```
sudo apt install openvpn -y 
```

- Criar o arquivo de configuração do servidor VPN chamado **server.conf** dentro de **/etc/openvpn/** e configura-lo dessa forma.
```
{
    dev tun 
    ifconfig 192.168.0.100	192.168.0.200
    secret /etc/openvpn/chave
    port 1194
    proto udp  
    comp-lzo
    verb 4
    keepalive 10 120
    persist-key
    persist-tun
    float
    cipher AES256
}
```

- Criar o arquivo de configuração do cliente VPN chamado **client.conf** dentro de **/etc/openvpn/** e configura-lo dessa forma.
```
{
    dev tun 
    ifconfig 192.168.0.200	192.168.0.100
    remote 100.29.171.248
    secret /etc/openvpn/chave
    port 1194
    proto udp  
    comp-lzo
    verb 4
    keepalive 10 120
    persist-key
    persist-tun
    float
    cipher AES256
}
```

- Gerar a chave para a VPN em **/etc/openvpn**.
```
openvpn –genkey –secret chave
```

- Enviar uma copia da chave para o cliente junto com o arquivo de configuração **cliente.conf**  (Obs: A chave deve estar no diretório informado no arquivo de configuração, nesse caso em /etc/openvpn/chave).

- Ligar o servidor VPN (Entre em **/etc/openvpn**).
```
sudo openvpn --config chave
```

- Conectar o cliente VPN ao servidor (Entre em **/etc/openvpn**).
```
sudo openvpn --config chave
```

**Desta forma a infraestrutura está funcionando!**<br/><br/>
**Para acessar a página desejada, o cliente precisará, primeiramente, se conectar à VPN; sem isso, o acesso será recusado.**<br/><br/>
**Uma vez conectado ao servidor VPN, ao digitar o IP 192.168.0.100 no navegador, o cliente será automaticamente redirecionado para a página desejada. O IP real da instância EC2, onde o projeto está hospedado, é camuflado pelo proxy reverso, enquanto o load balancer distribui as cargas de acordo com as requisições.**<br/><br/>
**Tanto o front-end quanto o back-end do projeto estão sendo executados via Docker, e os dados utilizados são armazenados e monitorados em um banco MySQL, que está em uma instância RDS na AWS.**

<img width=100% src="https://capsule-render.vercel.app/api?type=waving&color=d3d3d3&height=120&section=footer"/>
