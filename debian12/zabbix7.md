# Script em Shell Automatizando Zabbix em Container

Esse script ele instala o docker, depois cria o diretório conforme o padrão que está na documentação, logo após isso ele cria o docker-compose.yaml e lembrando que tem campos no arquivo do docker compose que devem ser preenchido de acordo com seu ambiente, tais como:

DB_SERVER_HOST: Colocar o IP ou DNS do seu Zabbix

ZBX_PASSIVESERVERS:Colocar o IP ou DNS do seu Zabbix

Você deve salvar esse Script que está abaixo com o nome que desejar, recomendo como zabbix.sh pois é em shellscript e não pode esquecer o ".sh", você deve dar permissão a esse script.

## Comando para criar o arquivo

```sh
nano zabbix.sh
```
## Comando para dar permissão ao script

```sh
chmod +x zabbix.sh
```

# Abaixo conteúdo do script

```sh
#!/bin/bash

# Instalar o sudo
apt-get install sudo

# Atualizar repositórios de sistema
sudo apt update -y && apt upgrade -y
apt list --upgradable

# Instalar dependências necessárias
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release

# Adicionar a chave GPG oficial do Docker
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Adicionar o repositório do Docker
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Atualizar a lista de pacotes novamente
sudo apt update

# Instalar o Docker e o Docker Compose
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose

# Verificar se a instalação do Docker foi bem-sucedida
if ! command -v docker &> /dev/null; then
    echo "Docker não instalado. Verifique os erros."
    exit 1
fi

# Verificar se a instalação do Docker Compose foi bem-sucedida
if ! command -v docker-compose &> /dev/null; then
    echo "Docker Compose não instalado. Verifique os erros."
    exit 1
fi

# Criar o diretório onde ficará o docker-compose.yaml
sudo mkdir -p /home/zabbix/

# Criar o arquivo docker-compose.yaml
cat <<EOF > /home/zabbix/docker-compose.yaml
services:
  zabbix-server:
    container_name: "zabbix-server"
    image: zabbix/zabbix-server-pgsql:alpine-7.0-latest
    restart: always
    ports:
      - "10051:10051"
    networks:
      zabbix7:
        ipv4_address: "172.18.0.2"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
    environment:
      ZBX_CACHESIZE: 256M
      ZBX_HISTORYCACHESIZE: 1024M
      ZBX_HISTORYINDEXCACHESIZE: 1024M
      ZBX_TRENDCACHESIZE: 1024M
      ZBX_VALUECACHESIZE: 1024M
      DB_SERVER_HOST: "172.18.0.4"
      DB_PORT: 5432
      POSTGRES_USER: "zabbix"
      POSTGRES_PASSWORD: "zabbix123"
      POSTGRES_DB: "zabbix_db"
    stop_grace_period: 30s
    labels:
      com.zabbix.description: "Zabbix server with PostgreSQL database support"
      com.zabbix.company: "Zabbix LLC"
      com.zabbix.component: "zabbix-server"
      com.zabbix.dbtype: "pgsql"
      com.zabbix.os: "alpine"

  zabbix-web-nginx-pgsql:
    container_name: "zabbix-web"
    image: zabbix/zabbix-web-nginx-pgsql:alpine-7.0-latest
    restart: always
    ports:
      - "8080:8080"
      - "8443:8443"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - ./cert/:/usr/share/zabbix/conf/certs/:ro
    networks:
      zabbix7:
        ipv4_address: "172.18.0.3"
    environment:
      ZBX_SERVER_HOST: "172.18.0.2"
      DB_SERVER_HOST: "172.18.0.4"
      DB_PORT: 5432
      POSTGRES_USER: "zabbix"
      POSTGRES_PASSWORD: "zabbix123*"
      POSTGRES_DB: "zabbix_db"
      ZBX_MEMORYLIMIT: "1024M"
    depends_on:
      - zabbix-server
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/ping"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 30s
    stop_grace_period: 10s
    labels:
      com.zabbix.description: "Zabbix frontend on Nginx web-server with PostgreSQL database support"
      com.zabbix.company: "Zabbix LLC"
      com.zabbix.component: "zabbix-frontend"
      com.zabbix.webserver: "nginx"
      com.zabbix.dbtype: "pgsql"
      com.zabbix.os: "alpine"

  zabbix-db-agent:
    container_name: "zabbix-agent"
    image: zabbix/zabbix-agent:alpine-7.0-latest
    depends_on:
      - zabbix-server
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /etc/timezone:/etc/timezone:ro
      - /run/docker.sock:/var/run/docker.sock
    environment:
      ZBX_HOSTNAME: "zabbix7"
      ZBX_SERVER_HOST: "172.18.0.2"
      ZBX_ENABLEREMOTECOMMANDS: "1"
    ports:
      - "10050:10050"
      - "31999:31999"
    networks:
      zabbix7:
        ipv4_address: "172.18.0.5"
    stop_grace_period: 5s

  db:
    container_name: "zabbix_db"
    image: postgres:16-bullseye
    restart: always
    volumes:
      - zbx_db16:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      zabbix7:
        ipv4_address: "172.18.0.4"
    environment:
      POSTGRES_USER: "zabbix"
      POSTGRES_PASSWORD: "zabbix123"
      POSTGRES_DB: "zabbix_db"

networks:
  zabbix7:
    driver: bridge
    ipam:
      config:
        - subnet: "172.18.0.0/16"

volumes:
  zbx_db16:
EOF

# Navegar para o diretório e criar os containers
cd /home/zabbix/

# Criar os containers
sudo docker compose up -d

# Verificar os containers em execução
sudo docker ps

# Para ver os logs de um container específico
# sudo docker logs nomedocontainer
``````

## Como Contribuir

Se você tem algo para contribuir, como novos templates, scripts ou guias de configuração, fique à vontade para enviar uma solicitação de pull. Contribuições são bem-vindas e apreciadas!

## Licença

Este repositório é fornecido sob a [Licença MIT](LICENSE). Sinta-se à vontade para usar, modificar e distribuir o conteúdo conforme necessário.

## Contato

Para perguntas, sugestões ou apenas para dizer olá, você pode entrar em contato com os mantenedores deste repositório através das issues ou por e-mail em fvcunhaa@gmail.com.
