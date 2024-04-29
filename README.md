**GABRIEL MACIEL DE MORAIS**

# Configuração do Nginx como Balanceador de Carga para Jenkins com SSL e Proxy Reverso

Este é um guia passo a passo sobre como configurar o Nginx como um balanceador de carga para o Jenkins, implementar a criptografia SSL para o domínio "jenkins.gabrielmoraislabs.com" e configurar um proxy reverso para encaminhar o tráfego para o Jenkins.

**Pré-requisitos**

- Um servidor Linux com acesso root


## Passo 1: Instalando e Configurando o Jenkins

### 1\.1. Instale o Jenkins no seu servidor Linux:

sudo apt update

sudo apt install default-jre

wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -

sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'

sudo apt update

sudo apt install Jenkins

### 1\.2. Inicie o Jenkins e habilite-o para iniciar na inicialização:

sudo systemctl start jenkins

sudo systemctl enable Jenkins

### 1\.3. Verifique se o Jenkins está em execução acessando **http://SEU\_IP\_SERVIDOR:8080** no seu navegador. Siga as instruções para concluir a instalação.

### 1\.4. Para executar o Jenkins em portas diferentes das padrão (geralmente 8080), você precisa especificar a porta durante a inicialização do Jenkins. Por exemplo, para executar o Jenkins na porta 8081:

sudo systemctl stop jenkins

sudo systemctl edit jenkins.service

Isso abrirá um editor de texto. Adicione as seguintes linhas:

[Service]

Environment="JENKINS\_PORT=8081"

Salve e feche o editor de texto. Em seguida, inicie o Jenkins:

sudo systemctl start Jenkins

### 1\.5. Abra as portas necessárias no firewall para permitir o tráfego do Jenkins. Por exemplo, se estiver usando o UFW, você pode abrir a porta 8080 com:

sudo ufw allow 8080


## Passo 2: Instalando e Configurando o Nginx**

### 2\.1. Instale o Nginx no seu servidor Linux:

sudo apt update

sudo apt install nginx

### 2\.2. Após a instalação, o Nginx deve estar em execução. Você pode verificar usando:

sudo systemctl status nginx

### 2\.3. Abra as portas necessárias no firewall para permitir o tráfego HTTP (80) e HTTPS (443). Por exemplo, usando UFW:

sudo ufw allow 'Nginx Full'

### 2\.4 Habilitar o Nginx para que start no boot da máquina, para garantir que o Nginx start automaticamente caso o servidor necessite de um restart:

sudo systemctl enable nginx


## Passo 3: Configurando o Nginx

### 3\.1. Crie um novo arquivo de configuração do server block do Nginx para o Jenkins. Crie o novo arquivo no caminho /etc/nginx/sites-available/your\_file. Neste teste foi dado o nome de jenkins ao arquivo:

sudo nano /etc/nginx/sites-available/jenkins

### 3\.2. Adicione o seguinte bloco de configuração para definir o upstream do Jenkins e o proxy reverso:


upstream jenkins {

  server localhost:8080;

  server localhost:8081;

  server localhost:8082;

}

server {

  listen 80;

  server\_name jenkins.gabrielmoraislabs.com;

  location / {

  proxy\_pass http://jenkins;

  proxy\_set\_header Host $host;

  proxy\_set\_header X-Real-IP $remote\_addr;

  proxy\_set\_header X-Forwarded-For $proxy\_add\_x\_forwarded\_for;

  proxy\_set\_header X-Forwarded-Proto $scheme;

  proxy\_redirect off;

  }

}

Salve e feche o arquivo de configuração do Nginx.

### 3\.3. Adicione um formato de log personalizado para incluir informações sobre o nome do servidor, o endereço do servidor upstream e a solicitação:

log\_format upstream\_log '$server\_name to $upstream\_addr [$request]';

Defina o log de acesso para usar o novo formato de log personalizado:

access\_log /var/log/nginx/access.log upstream\_log;

### 3\.4. Crie um link simbólico para o arquivo de configuração no diretório sites-enabled:

sudo ln -s /etc/nginx/sites-available/jenkins /etc/nginx/sites-enabled/

### 3\.5. Conferir a configuração do Nginx se está tudo correto:

sudo nginx -t

### 3\.6. Reinicie o Nginx para aplicar as alterações:

sudo systemctl restart nginx


## Passo 4: Instalando e Configurando o Certbot

### 4\.1. Instale o Certbot com suporte para Nginx:

sudo apt install python3-certbot-nginx

### 4\.2. Após a instalação, você pode verificar a versão do Certbot usando:

certbot –version

### 4\.3. Gerar e Instalar o Certificado SSL para o seu domínio (substitua **jenkins.gabrielmoraislabs.com** pelo seu domínio real):

sudo certbot --nginx -d jenkins.gabrielmoraislabs.com

Este comando irá: 

\- Detectar automaticamente as configurações do Nginx para o domínio especificado.

\- Solicitar permissão para gerar e instalar um certificado SSL. 

\- Configurar automaticamente o Nginx para usar o certificado SSL.


## Verificando os Logs

### Você pode visualizar os logs de acesso do Nginx para monitorar as solicitações encaminhadas para o Jenkins:

sudo tail -f /var/log/nginx/access.log


## Provas Visuais

![JENKINS1](https://github.com/Gabrielmmorais/Teste-Poder360-GabrielMorais-DEV-INFRA--TESTE1/assets/79981718/167a1075-9fae-417d-83a4-76d185566ffa)
*Instalação do jenkins dentro de uma EC2 micro apenas para o lab.*

![jenkins2](https://github.com/Gabrielmmorais/Teste-Poder360-GabrielMorais-DEV-INFRA--TESTE1/assets/79981718/61ef6f14-31e7-4a5b-a760-6ad46f26536f)
*Instalação do Nginx*

![jenkins3](https://github.com/Gabrielmmorais/Teste-Poder360-GabrielMorais-DEV-INFRA--TESTE1/assets/79981718/ff18653a-e9e1-4166-83a5-657b80e33c56)
*Jenkins rodando com https ativo e por meio de proxy reverso no domínio passando pelo NGINX para melhor segurança.*

![JENKINS4](https://github.com/Gabrielmmorais/Teste-Poder360-GabrielMorais-DEV-INFRA--TESTE1/assets/79981718/f0a00a44-4887-4b19-a005-6beb204c4008)
*Testes de conexão a aplicação por diferentes portas do Load Balancer.*



