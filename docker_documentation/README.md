**Documentação sobre Containers, Dockerfiles e Docker Compose**

## Índice

- [1. Introdução](#introdução)
- [2. Conceitos Fundamentais](#conceitos-fundamentais)
  - [2.1 Containers](#21-containers)
  - [2.2 Imagens](#22-imagens)
- [3. Dockerfile](#3-dockerfile)
  - [3.1 Componentes de um Dockerfile](#31-componentes-de-um-dockerfile)
- [4. Docker Compose](#4-docker-compose)
  - [4.1 Componentes do Docker Compose](#41-componentes-do-docker-compose)
- [5. Comandos mais utilizados do Docker e Docker Compose](#5-comandos-mais-utilizados-do-docker)
- [6. Configuração do repositório do Docker](#6-configuração-do-repositório-do-docker)
- [7. Referências](#7-referências)

## 1. Introdução

O Docker é uma plataforma que permite a criação, execução e gestão de containers. Containers são unidades padronizadas de software que empacotam código e todas as suas dependências, garantindo que a aplicação rode de forma consistente em qualquer ambiente.

## 2. Conceitos Fundamentais

### 2.1 Containers
O termo "container" é inspirado na indústria de transporte de cargas, onde containers físicos padronizados são usados para transportar mercadorias de forma eficiente entre diferentes meios de transporte e terminais. Analogamente, no desenvolvimento de software, os containers permitem que aplicações sejam movidas de um ambiente para outro (desenvolvimento, teste, produção) com consistência e sem preocupações com dependências específicas de cada ambiente.​

No contexto do desenvolvimento, esses ambientes isolados incluem tudo o necessário para executar uma aplicação (bibliotecas, dependências e arquivos de configuração, etc). Essa abordagem resolve o problema de inconsistência entre diferentes ambientes pelos quais uma solução de software transita, garantindo que o software funcione conforme esperado independentemente do ambiente.​

**Comparação entre Containers e Máquinas Virtuais**

Embora tanto containers quanto máquinas virtuais forneçam ambientes isolados para aplicações, eles diferem significativamente em sua arquitetura e desempenho:​

- **Máquinas Virtuais**: Cada VM inclui um sistema operacional completo, rodando sobre um hypervisor que emula o hardware subjacente. Isso resulta em maior consumo de recursos e tempos de inicialização mais longos.​

- **Containers**: Compartilham o kernel do sistema operacional host, criando ambientes isolados para aplicações sem a necessidade de um sistema operacional completo para cada instância. Isso os torna mais leves e permite tempos de inicialização mais rápidos.​

Essa leveza possibilita a execução de múltiplos containers em um único host sem a sobrecarga associada a várias VMs. Além disso, o isolamento dos containers é implementado por meio de **namespaces do Linux** ([saiba mais aqui](https://man7.org/linux/man-pages/man7/namespaces.7.html)), que permitem que cada container tenha sua própria visão dos recursos do sistema, como processos, rede e sistema de arquivos. ​



### 2.2 Imagens

Uma imagem Docker é um **modelo imutável** que serve como base para a criação de containers. Elas contêm todas as instruções necessárias para configurar o ambiente de execução da aplicação, incluindo o sistema operacional, dependências e configurações específicas. As imagens são construídas a partir de um arquivo chamado Dockerfile e podem ser armazenadas em repositórios, como o Docker Hub. Uma vez criadas, as imagens garantem que os containers derivados delas sejam consistentes em diferentes ambientes.​

**Estrutura de Camadas das Imagens Docker**

Um dos pontos mais interessante é que as imagens Docker são compostas por **múltiplas camadas empilhadas**, onde cada camada representa uma modificação ou adição feita durante o processo de construção da imagem. Cada instrução no Dockerfile que altera o sistema de arquivos (como RUN, COPY e ADD) cria uma nova camada. Essas camadas são imutáveis e somente leitura. Quando um container é iniciado a partir de uma imagem, uma camada adicional de leitura/escrita é adicionada no topo das camadas existentes, permitindo que o container faça alterações sem modificar a imagem base. ​

Uma vez que as imagens compartilham camadas em comum, isso permite vários 

    Reutilização: Se várias imagens compartilham camadas comuns, o Docker reutiliza essas camadas, economizando espaço em disco e acelerando o processo de construção.​

    Eficiência no Cache: Durante a construção de uma imagem, se uma camada não sofreu alterações, o Docker pode reutilizar o cache existente, tornando o processo mais rápido.​

    Distribuição Eficiente: Ao transferir imagens entre sistemas, apenas as camadas ausentes ou modificadas precisam ser enviadas, otimizando o uso de banda e tempo.​

## 3. Dockerfile

O Dockerfile é um arquivo de texto que contém uma série de instruções utilizadas pelo Docker para construir uma imagem. Essas instruções especificam o sistema operacional base, as dependências necessárias, configurações do ambiente, comandos de inicialização e outros aspectos essenciais para a configuração do ambiente de execução da aplicação. A utilização de um `Dockerfile` permite a automação e reprodução consistente da construção de imagens, facilitando o desenvolvimento e a implantação de aplicações. 
Cada instrução no Dockerfile adiciona uma nova camada à imagem, permitindo a construção de imagens de forma declarativa e reproduzível.​

**Principais Instruções do Dockerfile:**
    - FROM: Define a imagem base a partir da qual a nova imagem será construída.​
    - RUN: Executa comandos no ambiente da imagem durante o processo de construção.​
    - COPY / ADD: Copia arquivos do sistema de arquivos do host para o sistema de arquivos da imagem.​
    - WORKDIR: Define o diretório de trabalho dentro do container.​
    - CMD / ENTRYPOINT: Especifica o comando ou script que será executado quando o container for iniciado.​
    - EXPOSE: Informa quais portas a aplicação dentro do container irá escutar.​


### 3.1 Componentes de um Dockerfile

Um `Dockerfile` é composto por diversas instruções, entre as quais:

- **FROM**: Define a imagem base a partir da qual a nova imagem será construída. Por exemplo, `FROM ubuntu:latest` especifica que a construção deve começar a partir da última versão da imagem Ubuntu disponível.

- **RUN**: Executa comandos durante a construção da imagem. Esses comandos são frequentemente utilizados para instalar pacotes e configurar o ambiente. Por exemplo, `RUN apt-get update && apt-get install -y python3` atualiza os repositórios e instala o Python 3.

- **COPY** / **ADD**: Copiam arquivos do sistema de arquivos do host para o sistema de arquivos da imagem. `COPY` é usado para copiar arquivos locais, enquanto `ADD` pode também lidar com URLs remotas e arquivos compactados. Por exemplo, `COPY . /app` copia todos os arquivos do diretório atual para o diretório `/app` na imagem.

- **WORKDIR**: Define o diretório de trabalho dentro do container. Com `WORKDIR /app`, qualquer comando subsequente será executado dentro do diretório `/app`.

- **CMD** / **ENTRYPOINT**: Especificam o comando a ser executado quando o container é iniciado. `CMD` define o comando padrão, que pode ser sobrescrito, enquanto `ENTRYPOINT` define o comando que será sempre executado. Por exemplo, `CMD ["python3", "app.py"]` define que o script `app.py` será executado com o Python 3 ao iniciar o container.

- **EXPOSE**: Informa quais portas o container irá expor em tempo de execução. Por exemplo, `EXPOSE 80` indica que o container irá escutar na porta 80.

Pensando num projeto prático com a estrutura a seguir, teremos um dockerfile para o front e backend. 
```bash
meu-projeto/
├── backend/
│   ├── Dockerfile
│   └── app.js
├── frontend/
│   ├── Dockerfile
│   └── index.html
└── docker-compose.yml
```
De forma geral, a estrutura dos dockerfiles são tal como a seguir: 

```dockerfile
# Usa a imagem oficial do Node.js como base
FROM node:14

# Define o diretório de trabalho dentro do container
WORKDIR /usr/src/app

# Copia os arquivos do backend para o diretório de trabalho
COPY . .

# Instala as dependências
RUN npm install

# Expõe a porta em que a aplicação será executada
EXPOSE 3000

# Comando para iniciar a aplicação
CMD ["node", "app.js"]

```

E o frontend 

```dockerfile
# Usa a imagem oficial do Nginx como base
FROM nginx:alpine

# Copia o arquivo index.html para o diretório padrão do Nginx
COPY index.html /usr/share/nginx/html/index.html

# Expõe a porta padrão do Nginx
EXPOSE 80
```


## 4. Docker Compose

O Docker Compose é uma ferramenta que permite definir e gerenciar aplicações multicontainer. Ele utiliza um arquivo `docker-compose.yml` para descrever os serviços, redes e volumes necessários para executar a aplicação. A partir dele é possível iniciar e orquestrar todos os serviços de uma aplicação com poucos comandos (como `docker-compose up` e `docker-compose down`), simplificando o processo de desenvolvimento e implantação. 

### 4.1 Componentes do Docker Compose

De modo geral, um arquivo `docker-compose.yml` define:
- **version**: Versão do Compose utilizada.
- **services**: Define os containers a serem executados.
- **volumes**: Permite a persistência de dados.
- **networks**: Configuração de redes para comunicação entre containers.


Para esclarecer na prática, considere que nossa aplicação tem o seguinte cenário

```bash
meu-projeto/
├── backend/
│   ├── Dockerfile
│   └── app.js
├── frontend/
│   ├── Dockerfile
│   └── index.html
└── docker-compose.yml
```

A estrutura do nosso `docker-compose.yml` pode ser do tipo 

```yml
version: "3.9"  # Define a versão do Docker Compose a ser utilizada.

services:
  backend:
    build: ./backend  # Especifica o caminho para o diretório que contém o Dockerfile do backend.
    ports:
      - "3000:3000"  # Mapeia a porta 3000 do host para a porta 3000 do container, permitindo acesso externo.
    networks:
      - app-network  # Conecta o serviço à rede personalizada 'app-network'.
    depends_on:
      - db  # Garante que o serviço 'db' seja iniciado antes do 'backend'
    environment:
      DB_HOST: db  # Nome do serviço do banco de dados
      DB_PORT: 5432  # Porta padrão do PostgreSQL
      DB_USER: example_user  # Usuário do banco de dados
      DB_PASSWORD: example_password  # Senha do banco de dados
      DB_NAME: example_db  # Nome do banco de dados

  frontend:
    build: ./frontend  # Especifica o caminho para o diretório que contém o Dockerfile do frontend.
    ports:
      - "80:80"  # Mapeia a porta 80 do host para a porta 80 do container, permitindo acesso externo.
    networks:
      - app-network  # Conecta o serviço à rede personalizada 'app-network'.
    depends_on:
      - backend  # Indica que o serviço 'frontend' depende do serviço 'backend', garantindo que o backend seja iniciado antes.
  db:
    image: postgres:latest  # Utiliza a imagem oficial do PostgreSQL
    container_name: postgres_container  # Nomeia o container como 'postgres_container'
    ports:
      - "5432:5432"  # Mapeia a porta 5432 do host para a porta 5432 do container
    networks:
      - app-network  # Conecta o serviço à rede 'app-network'
    environment:
      POSTGRES_USER: example_user  # Define o usuário do PostgreSQL
      POSTGRES_PASSWORD: example_password  # Define a senha do PostgreSQL
      POSTGRES_DB: example_db  # Cria o banco de dados 'example_db'
    volumes:
      - postgres-data:/var/lib/postgresql/data  # Persiste os dados do banco

networks:
  app-network:
    driver: bridge  # Define a rede 'app-network' com o driver 'bridge'

volumes:
  postgres-data:
    driver: local  # Define o volume 'postgres-data' com o driver 'local'

```
**Importante!**
É recomendável utilizar um arquivo `.env` para armazenar variáveis sensíveis, como credenciais de banco de dados, e referenciá-las no docker-compose.yml. Isso melhora a segurança e permite maior flexibilidade à possíveis mudanças.

--- 
## 5. Comandos mais utilizados do Docker
Quando queremos fazer o gerenciamento de containers individualmente, os comandos a seguir são alguns dos mais utilizados.

### **Gerenciamento de Imagens e Containers**  

```sh
# Baixa uma imagem do Docker Hub (ou outro repositório)
docker pull <imagem>:<tag>  # Exemplo: docker pull nginx:latest

# Lista todas as imagens baixadas localmente
docker images

# Remove uma imagem localmente
docker rmi <imagem_id>  # Exemplo: docker rmi 123abc456def
```

### **Execução e Gerenciamento de Containers**  

```sh
# Cria e inicia um container com base em uma imagem
docker run -d --name meu_container -p 8080:80 nginx  

# Lista todos os containers em execução
docker ps  

# Lista todos os containers (incluindo os parados)
docker ps -a  

# Para um container em execução
docker stop <container_id>  # Exemplo: docker stop abc123

# Reinicia um container
docker restart <container_id>  

# Remove um container
docker rm <container_id>  
```

### **Inspecionar e Interagir com Containers**  

```sh
# Exibe os logs de um container - importante para ver se tudo ocorreu bem
docker logs <container_id>  

# Acessa o terminal de um container em execução
docker exec -it <container_id> sh  
docker exec -it <container_id> bash  

# Exibe os detalhes do container (configuração, IP, etc.)
docker inspect <container_id>  

# Monitora os recursos consumidos pelos containers em tempo real
docker stats  
```

### **Gerenciamento de Volumes e Redes**  

```sh
# Lista os volumes criados no Docker
docker volume ls  

# Remove um volume
docker volume rm <volume_name>  

# Lista as redes disponíveis no Docker
docker network ls  

# Cria uma nova rede
docker network create minha_rede  

# Conecta um container a uma rede
docker network connect minha_rede <container_id>  

# Remove uma rede
docker network rm minha_rede  
```

### **Build e Manutenção de Imagens**  

```sh
# Constrói uma nova imagem a partir de um Dockerfile 
docker build -t minha_imagem .  

# Remove todas as imagens não utilizadas
docker image prune -a  
```

---

## **Comandos Docker Compose Mais Utilizados**  
Quando temos vários containers e não queremos inicializá-los manualmente, podemos fazer a orquestração utilizando o docker compose. Os comandos listados a seguir são alguns dos mais comuns no dia a dia. 

### **Gerenciamento de Serviços**  

```sh
# Inicia todos os serviços definidos no docker-compose.yml
docker compose up -d  # -d indica modo detached (não trava o terminal)

# Para todos os serviços em execução
docker compose down  

# Para um serviço específico
docker compose stop <ID container>  

# Reinicia um serviço específico
docker compose restart <ID container>  
```

### **Monitoramento e Logs**  

```sh
# Lista os serviços em execução
docker compose ps  

# Exibe os logs de um serviço específico
docker compose logs <ID container>  

# Exibe logs em tempo real
docker compose logs -f <ID container>  
```


### **Build e Manutenção**  

```sh
# Reconstrói os serviços do Docker Compose sem usar cache
docker compose build --no-cache  

# Remove containers, volumes e redes associadas ao Compose
docker compose down --volumes --remove-orphans  

# Lista todas as imagens utilizadas no Docker Compose
docker compose images  
```


## 6. Configuração do repositório do Docker

Antes de instalar o Docker pela primeira vez, você precisa configurar o repositório `apt` do Docker.

### 6.1 Adicionar a chave GPG oficial do Docker

Execute os seguintes comandos para adicionar a chave GPG do Docker:

```
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### 6.2. Adicionar o repositório Docker ao APT

Agora, adicione o repositório Docker ao APT. O comando abaixo adiciona o repositório baseado na versão do Ubuntu que você está usando:

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

### 6.3. **Instalar os pacotes do Docker**

Agora, instale o Docker Engine e os pacotes relacionados com o seguinte comando:

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 6.4 **Verificar se a instalação foi bem-sucedida**

Após a instalação, verifique se o Docker foi instalado corretamente rodando o comando:

```
sudo docker run hello-world

```

### 6.5 **Uso do Docker sem root (opcional)**
Por padrão, você precisa usar sudo para rodar comandos do Docker, caso queira rodar comandos do Docker sem precisar do sudo, adicione seu usuário ao grupo docker com os seguinte comando:
```
sudo usermod -aG docker $USER
``` 

(Saia e entre novamente no sistema para aplicar a mudança.)


## 7. Referências
- Documentação oficial: https://docs.docker.com/reference/
- Sobre namespaces no linux: https://man7.org/linux/man-pages/man7/namespaces.7.html
- Tutorial pipeline de docker compose: https://github.com/daniil/full-stack-js-docker-tutorial
- Instalação docker no Ubuntu: https://docs.docker.com/engine/install/ubuntu/
