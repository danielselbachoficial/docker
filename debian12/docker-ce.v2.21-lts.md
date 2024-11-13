# Script em Shell Automatizando a instalação do Docker

Esse script ele instala o docker, depois cria o diretório conforme o padrão que está na documentação, logo após isso ele cria o docker-compose.yaml e lembrando que tem campos no arquivo do docker compose que devem ser preenchido de acordo com seu ambiente, tais como:

Você deve salvar esse Script que está abaixo com o nome que desejar, recomendo como docker-ce.v2.21.sh pois é em shellscript e não pode esquecer o ".sh", você deve dar permissão a esse script.

## Comando para criar o arquivo

```sh
nano docker-ce.v2.21.sh
```
## Comando para dar permissão ao script

```sh
chmod +x docker-ce.v2.21.sh
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

#!/bin/bash

# Atualizando o sistema e instalando pacotes necessários
apt update -y
apt install -y ca-certificates curl gnupg lsb-release

# Adicionando chave GPG do repositório Docker
mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Adicionando repositório Docker para Debian 12
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# Atualizando índices do APT e instalando Docker CE
apt update -y
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Configurando Docker para escutar na porta 15600 e aceitar apenas conexões locais e do IP especificado
mkdir -p /etc/systemd/system/docker.service.d
cat <<EOF | sudo tee /etc/systemd/system/docker.service.d/override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H fd:// -H tcp://0.0.0.0:15600
EOF

# Habilitando e reiniciando o serviço Docker
systemctl daemon-reload
systemctl enable docker
systemctl restart docker

# Adicionando o usuário atual ao grupo Docker para evitar necessidade de sudo
usermod -aG docker $USER

# Exibindo versão do Docker para verificar instalação
docker --version

echo "Instalação do Docker CE 23.0 LTS concluída com segurança e porta 15600 configurada para aceitar conexões somente do IP 138.97.35.201."
echo "É recomendável sair e fazer login novamente para aplicar as permissões de grupo."
``````
