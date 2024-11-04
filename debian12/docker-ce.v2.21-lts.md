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

#########################################################################################
# DANIEL S. FIGUEIRÓ                                                                    #
# IT CONSULTANT                                                                         #
# LINKEDIN: https://www.linkedin.com/in/danielselbachtech/                              #
# SCRIPT V.: 1.0 - Docker CE v2.21                                                      #
#########################################################################################

# Instalar o sudo
apt-get install sudo

# Atualizar repositórios de sistema
sudo apt update
sudo apt upgrade -y
apt list --upgradable

# Instalar pacotes essenciais
sudo apt install curl wget zip git ufw -y

# Escolher entre Apache ou Nginx
echo "Escolha o servidor web: 1 - Apache | 2 - Nginx"
read -r webserver_choice

if [ "$webserver_choice" -eq 1 ]; then
    echo "Instalando Apache..."
    sudo apt install apache2 -y
    sudo a2enmod rewrite
    sudo systemctl enable apache2
elif [ "$webserver_choice" -eq 2 ]; then
    echo "Instalando Nginx..."
    sudo apt install nginx -y
    sudo systemctl enable nginx
else
    echo "Escolha inválida! Encerrando o script."
    exit 1
fi

# Instalar e configurar MariaDB
sudo apt install mariadb-server mariadb-client -y
sudo systemctl enable mariadb

# Configuração segura do MariaDB
sudo mysql_secure_installation <<EOF
n
y
your_password_here
y
y
y
y
EOF

# Cria banco de dados e usuário para o phpIPAM
sudo mysql -u root -p <<EOF
CREATE DATABASE php_ipam;
GRANT ALL ON php_ipam.* TO 'phpipam'@'localhost' IDENTIFIED BY 'phpipamadmin';
FLUSH PRIVILEGES;
EXIT;
EOF

# Instalar componentes PHP necessários
sudo apt install php php-fpm php-curl php-mysql php-gmp php-mbstring php-xml -y
sudo systemctl enable php-fpm

# Verificar versão do PHP para ajustar Nginx
PHP_VERSION=$(php -r 'echo PHP_MAJOR_VERSION.".".PHP_MINOR_VERSION;')

# Baixar e configurar phpIPAM
sudo git clone https://github.com/phpipam/phpipam.git /var/www/html/phpipam
cd /var/www/html/phpipam || exit
sudo git checkout "$(git tag --sort=v:tag | tail -n1)"
sudo chown -R www-data:www-data /var/www/html/phpipam
sudo chmod -R 755 /var/www/html/phpipam

# Configurar o arquivo config.php
sudo cp /var/www/html/phpipam/config.dist.php /var/www/html/phpipam/config.php
sudo bash -c "cat > /var/www/html/phpipam/config.php" <<EOF
<?php
\$db['host'] = '127.0.0.1';
\$db['user'] = 'phpipam';
\$db['pass'] = 'phpipamadmin';
\$db['name'] = 'php_ipam';
\$db['port'] = 3306;

define('BASE' , '/phpipam/');
EOF

# Configuração do Nginx (se escolhido)
if [ "$webserver_choice" -eq 2 ]; then
    sudo bash -c "cat > /etc/nginx/sites-available/phpipam" <<EOF
server {
    listen 80;
    server_name _;

    root /var/www/html/phpipam;
    index index.php index.html index.htm;

    location / {
        try_files \$uri \$uri/ /index.php;
    }

    location ~ \.php\$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php${PHP_VERSION}-fpm.sock;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    error_log /var/log/nginx/phpipam_error.log;
    access_log /var/log/nginx/phpipam_access.log;
}
EOF
    sudo ln -s /etc/nginx/sites-available/phpipam /etc/nginx/sites-enabled/
    sudo systemctl reload nginx
fi

# Testar permissões e corrigir se necessário
echo "Ajustando permissões para phpIPAM..."
sudo chown -R www-data:www-data /var/www/html/phpipam
sudo find /var/www/html/phpipam -type d -exec chmod 755 {} \;
sudo find /var/www/html/phpipam -type f -exec chmod 644 {} \;

# Testar serviços
if [ "$webserver_choice" -eq 1 ]; then
    sudo systemctl restart apache2
    sudo systemctl status apache2
elif [ "$webserver_choice" -eq 2 ]; then
    sudo systemctl restart nginx
    sudo systemctl status nginx
fi
sudo systemctl restart mariadb
sudo systemctl status mariadb

echo "Instalação concluída! Acesse seu phpIPAM via http://<seu_ip>/phpipam"
``````

## Como Contribuir

Se você tem algo para contribuir, como novos templates, scripts ou guias de configuração, fique à vontade para enviar uma solicitação de pull. Contribuições são bem-vindas e apreciadas!

## Licença

Este repositório é fornecido sob a [Licença MIT](LICENSE). Sinta-se à vontade para usar, modificar e distribuir o conteúdo conforme necessário.

## Contato

Para perguntas, sugestões ou apenas para dizer olá, você pode entrar em contato com os mantenedores deste repositório através das issues ou por e-mail em fvcunhaa@gmail.com.
