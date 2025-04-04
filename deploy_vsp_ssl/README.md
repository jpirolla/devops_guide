# Deploy de Aplicação Fullstack no VPS com Docker e Certbot para SSL 


## Índice  
1. [Introdução](#1-introdução)  
2. [Instalação do Docker no VPS](#2-instalação-do-docker-no-vps)  
3. [Configuração do Nginx como Proxy Reverso](#3-nginx)  
   - [O que é o Nginx?](#31-o-que-é-o-nginx)  
   - [O que é um Proxy Reverso?](#32-o-que-é-um-proxy-reverso)  
   - [Configuração do Nginx como Proxy Reverso](#33-configuração-do-nginx-como-proxy-reverso)  
4. [Configuração de Domínio e SSL com Certbot](#4-configuração-de-domínio-e-ssl-com-certbot)  
   - [Configuração do Certbot no docker-compose.yml](#41-configuração-do-certbot-no-docker-composeyml)  
   - [Configuração do Nginx para SSL](#42-configuração-do-nginx-para-ssl)  
5. [Testando e Reiniciando o Nginx e Docker Compose](#5-testando-e-reiniciando-o-nginx-e-docker-compose)  
6. [Referências](#6-referências)  

---

## 1. Introdução

Este documento descreve o processo de deploy de uma aplicação fullstack em um VPS, utilizando Docker para encapsular a aplicação, Nginx como proxy reverso e Certbot para gerenciamento de certificados SSL. 
O objetivo é garantir um ambiente seguro, escalável e de fácil replicação, reduzindo a complexidade da configuração manual e tornando o processo de deploy mais eficiente.

---

## 2. Instalação do Docker no VPS

Execute os seguintes comandos para instalar o Docker:
```bash
sudo apt update && sudo apt install -y docker.io
```

Verifique a instalação:

```bash
sudo systemctl enable --now docker
sudo docker --version
```

Caso queira utilizar o `docker-compose`, instale com:
```bash
sudo apt install -y docker-compose
```

---

## 3. Nginx
### 3.1 O que é o nginx?
O Nginx é um servidor web amplamente utilizado, que também pode atuar como balanceador de carga, cache e proxy reverso. Ele se destaca pela eficiência no tratamento de conexões, pois gerencia requisições HTTP de forma assíncrona e baseada em eventos. Diferentemente do Apache, que utiliza um modelo orientado a processos, o Nginx permite que um único processo gerencie múltiplas requisições simultaneamente, otimizando a performance.

No modelo do Nginx, temos:
- Processo Principal: Cria e gerencia os processos trabalhadores (worker processes).
- Processos Trabalhadores (Workers): Cada worker pode lidar com múltiplas requisições de forma assíncrona, utilizando multiplexação de I/O, garantindo alta escalabilidade e eficiência.

### 3.2 O que é um Proxy Reverso?

Um proxy reverso é um servidor intermediário que recebe requisições de clientes e as encaminha para um ou mais servidores backend, retornando as respostas aos clientes. Diferente de um proxy tradicional, que representa os clientes ao acessar recursos externos, o proxy reverso representa os servidores internos, protegendo-os e otimizando sua comunicação com os clientes.

Quando uma requisição é feita, em vez de ser enviada diretamente ao servidor de aplicação ou ao servidor web, ela passa primeiro pelo proxy reverso, que pode desempenhar diversas funções, como:
- Encaminhamento de requisições para diferentes servidores backend.
- Balanceamento de carga, distribuindo requisições entre vários servidores para melhorar a performance.
- Cache de conteúdo, reduzindo o número de requisições diretas ao backend e melhorando a velocidade de resposta.
- Redirecionamentos e reescrita de URLs, facilitando a administração de aplicações.
- Aprimoramento de segurança, ocultando a estrutura real dos servidores e protegendo contra ataques.

No nosso caso, atualmente, utilizamos apenas a funcionalidade de encaminhar as requisições e melhorar a segurança do servidor.

### 3.3 Configuração do Nginx como Proxy Reverso

Instale o Nginx. Caso vários usuários usem o nginx, é interessante instalar globalmente.
```bash
sudo apt update && sudo apt install -y nginx
```

Permita o tráfego HTTP e HTTPS:
```bash
sudo ufw allow 'Nginx Full'
```

Crie um arquivo de configuração para o frontend:
```bash
sudo nano /etc/nginx/sites-available/frontend
```

Adicione:
```nginx
server {
    listen 80;
    server_name seuDominio.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
Crie um para o backend:
```bash
sudo nano /etc/nginx/sites-available/backend
```

Adicione:
```nginx
server {
    listen 80;
    server_name api.seuDominio.com;

    location / {
        proxy_pass http://localhost:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Ative as configurações:
```bash
sudo ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/backend /etc/nginx/sites-enabled/
```

Reinicie o Nginx:
```bash
sudo systemctl restart nginx
```

## 4. Configuração de Domínio e SSL com Certbot

O Certbot é uma ferramenta gratuita e de código aberto usada para obter e renovar automaticamente certificados SSL/TLS da autoridade certificadora Let’s Encrypt. Esses certificados garantem que sua aplicação utilize HTTPS, protegendo a comunicação entre o servidor e os usuários. O certificado será configurado no Dockerfile para garantir a segurança das comunicações.
Configuração do Certbot no docker-compose.yml

Adicionamos a seguinte estrutura ao docker-compose.yml para configurar o Certbot:

```yml
services:
  # outros serviços como front e backend
  certbot:
    image: certbot/certbot
    container_name: certbot
    volumes:
      - /etc/letsencrypt:/etc/letsencrypt
      - /var/www/certbot:/var/www/certbot
    command: certonly --webroot -w /var/www/certbot --keep-until-expiring --email ${CERTBOT_EMAIL} -d ${CERTBOT_DOMAIN1} -d ${CERTBOT_DOMAIN2} --agree-tos
```

**ATENÇÃO**: Certifique-se de que todas as variáveis de ambiente necessárias (CERTBOT_EMAIL, CERTBOT_DOMAIN1, CERTBOT_DOMAIN2) estejam devidamente definidas no arquivo .env.

Após atualizar o yml, vamos reiniciar os contêineres e verificar os logs do serviço:

```bash
sudo docker compose logs certbot
```

Se tudo estiver correto, você verá a mensagem: *"Successfully received certificate"*

Agora, copie o caminho da chave privada exibido no campo "key saved at" e modifique o arquivo de configuração do Nginx para utilizar o certificado SSL.

### 4.1 Configuração do Nginx

Abaixo está a configuração do Nginx para redirecionar automaticamente para HTTPS e utilizar os certificados SSL emitidos pelo Certbot.

**Configuração para o Frontend (nginx/frontend.conf)**

```conf
server {
    listen 80;
    server_name seuDominioFrontend.com;

    location ~ /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    # Redirecionamento automático para HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name seuDominioFrontend.com;

    # exemplo: /etc/letsencrypt/live/seuDominioFrontend.com-0001/fullchain.pem
    ssl_certificate /etc/letsencrypt/live/seuDominioFrontend.com-key/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/seuDominioFrontend.com-key/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:suaPortaAplicacao;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Configuração para o Backend (nginx/backend.conf)**
```conf
server {
    listen 80;
    server_name seuDominioBackend.com;

    location ~ /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    # Redirecionamento automático para HTTPS
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name seuDominioBackend.com;

    ssl_certificate /etc/letsencrypt/live/seuDominioBackend.com-key/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/seuDominioBackend.com-key/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:suaPortaAplicacao-back;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

## 5. Testando e Reiniciando o Nginx e Docker compose

Após configurar o Nginx, teste a sintaxe do arquivo para garantir que não haja erros:

```bash
sudo nginx -t
```

Se o teste for bem-sucedido, reinicie o serviço:
```bash
sudo systemctl restart nginx
```

**Atualizando o docker-compose.yml**

Agora, alteramos o docker-compose.yml para incluir os serviços do MySQL, backend, frontend, Certbot e Watchtower(opcional):

```yml
services:
  mysql:
    image: mysql:8.0
    container_name: mysql_db
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD}
      - MYSQL_DATABASE=${MYSQL_DATABASE}
      - MYSQL_USER=${MYSQL_USER} 
      - MYSQL_PASSWORD=${MYSQL_PASSWORD}
    ports: 
      - "3305:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  backend:
    image: fabrice45/school-backend
    restart: always
    ports:
      - "5001:5001" #verificar aqui
    environment:
      - DB_URL=${DB_URL}
      - DB_USERNAME=${MYSQL_USER}
      - DB_PASSWORD=${MYSQL_PASSWORD}
    depends_on:
      - mysql

  frontend:
    image: fabrice45/school-frontend
    restart: always
    ports:
      - "5002:5002"
    depends_on:
      - backend

  certbot:
    image: certbot/certbot
    container_name: certbot
    volumes:
      - /etc/letsencrypt:/etc/letsencrypt
      - /var/www/certbot:/var/www/certbot
    command: certonly --webroot -w /var/www/certbot --keep-until-expiring --email ${CERTBOT_EMAIL} -d ${CERTBOT_DOMAIN1} -d ${CERTBOT_DOMAIN2} --agree-tos

  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - WATCHTOWER_CLEANUP=true
      - DOCKER_HUB_USERNAME=${DOCKER_HUB_USERNAME}
      - DOCKER_HUB_PASSWORD=${DOCKER_HUB_ACCESS_TOKEN}
    command: --interval 43200  # verifica atualização a cada 12 horas

volumes:
  mysql_data:
  certbot-ssl:
  certbot-challenge:
```


Agora, suba os contêineres novamente:

```bash
sudo docker compose up -d
```

Isso garantirá que todos os serviços sejam iniciados corretamente e ao acessar seu domínio, teremos um redirecionamento para HTTPS. 



## 6. Referências
- [How to Deploy Full Stack Spring Boot App on VPS with Docker, NGINX, SSL, & GH CI/CD](https://www.youtube.com/watch?v=RgZFoC0QHbg&t=1991s)
- [Docker Documentation](https://docs.docker.com/)
- [Nginx Documentation](https://nginx.org/en/docs/)
- [Certbot Documentation](https://certbot.eff.org/)
- [GitHub Actions](https://docs.github.com/en/actions)

