🚀 INSTALAÇÃO MANUAL COMPLETA - EQUIPECHAT
Ubuntu Server 22.04 LTS
📋 PRÉ-REQUISITOS
bash
# Acessar servidor
ssh seu_usuario@ip_do_servidor

# Domínios (DEVEM apontar para o IP do servidor)
# backend: api.seusite.com
# frontend: app.seusite.com
PARTE 1: CONFIGURAÇÃO INICIAL
1.1 Criar usuário deploy
bash
# Criar usuário
sudo useradd -m -s /bin/bash deploy

# Definir senha (ex: Deploy@2024)
sudo passwd deploy

# Adicionar ao grupo sudo
sudo usermod -aG sudo deploy

# Testar acesso
sudo su - deploy
exit
1.2 Atualizar sistema
bash
sudo apt update
sudo apt upgrade -y
sudo apt autoremove -y
1.3 Instalar dependências básicas
bash
sudo apt install -y curl wget git unzip zip htop \
  net-tools software-properties-common \
  apt-transport-https ca-certificates gnupg \
  lsb-release build-essential
PARTE 2: INSTALAÇÃO DO NODE.JS
2.1 Instalar Node.js 20
bash
# Adicionar repositório
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo bash -

# Instalar
sudo apt install -y nodejs

# Verificar
node --version
npm --version

# Atualizar npm
sudo npm install -g npm@latest
PARTE 3: INSTALAÇÃO DO POSTGRESQL
3.1 Instalar PostgreSQL
bash
# Instalar
sudo apt install -y postgresql postgresql-contrib

# Iniciar serviço
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Verificar
sudo systemctl status postgresql
psql --version
3.2 Configurar PostgreSQL
bash
# Definir senha do postgres (use a mesma senha do .env)
sudo -u postgres psql -c "ALTER USER postgres WITH PASSWORD 'sua_senha_aqui';"

# Criar banco para sua instância (ex: empresa)
sudo -u postgres createdb nome_da_empresa

# Criar usuário do banco
sudo -u postgres psql -c "CREATE USER nome_da_empresa WITH SUPERUSER PASSWORD 'sua_senha_aqui';"

# Verificar
sudo -u postgres psql -c "\l" | grep nome_da_empresa
PARTE 4: INSTALAÇÃO DO DOCKER
4.1 Instalar Docker
bash
# Dependências
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# Adicionar chave GPG
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# Adicionar repositório
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io

# Adicionar usuário ao grupo docker
sudo usermod -aG docker deploy

# Iniciar Docker
sudo systemctl start docker
sudo systemctl enable docker

# Verificar
docker --version
docker-compose --version
4.2 Criar container Redis
bash
# Criar Redis (use a porta que definiu, ex: 5000)
docker run --name redis-nome_da_empresa \
  -p 5000:6379 \
  --restart always \
  --detach redis \
  redis-server --requirepass sua_senha_aqui

# Verificar
docker ps | grep redis
PARTE 5: INSTALAÇÃO DO PM2
5.1 Instalar PM2 global
bash
sudo npm install -g pm2

# Configurar para iniciar com o sistema (como usuário deploy)
sudo su - deploy
pm2 startup systemd
exit

# Verificar
pm2 --version
PARTE 6: INSTALAÇÃO DO NGINX
6.1 Instalar Nginx
bash
sudo apt install -y nginx

# Remover configuração padrão
sudo rm /etc/nginx/sites-enabled/default

# Configurar limite de upload
echo "client_max_body_size 100M;" | sudo tee /etc/nginx/conf.d/equipechat.conf

# Testar configuração
sudo nginx -t

# Reiniciar
sudo systemctl restart nginx
sudo systemctl enable nginx
PARTE 7: INSTALAÇÃO DO CERTBOT (SSL)
7.1 Instalar Certbot via Snap
bash
# Instalar snapd
sudo apt install -y snapd
sudo snap install core
sudo snap refresh core

# Instalar certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

# Verificar
certbot --version
PARTE 8: INSTALAÇÃO DAS DEPENDÊNCIAS DO PUPPETEER
8.1 Instalar bibliotecas necessárias
bash
sudo apt install -y libxshmfence-dev libgbm-dev wget unzip \
  fontconfig locales gconf-service libasound2 libatk1.0-0 \
  libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 \
  libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 \
  libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 \
  libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 \
  libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 \
  libxtst6 ca-certificates fonts-liberation libappindicator1 \
  libnss3 lsb-release xdg-utils libgbm1 libxkbcommon0
PARTE 9: CLONAR E CONFIGURAR O PROJETO
9.1 Clonar repositório (como usuário deploy)
bash
# Trocar para usuário deploy
sudo su - deploy

# Clonar
git clone https://github.com/seu-usuario/seu-repo.git /home/deploy/nome_da_empresa/

# Entrar no diretório
cd /home/deploy/nome_da_empresa
PARTE 10: CONFIGURAR E COMPILAR BACKEND
10.1 Criar arquivo .env do backend
bash
cd /home/deploy/nome_da_empresa/backend

# Criar .env
cat > .env << 'END'
NODE_ENV=
BACKEND_URL=https://api.seusite.com
FRONTEND_URL=https://app.seusite.com
PROXY_PORT=443
PORT=4000

DB_HOST=localhost
DB_DIALECT=postgres
DB_USER=nome_da_empresa
DB_PASS=sua_senha_aqui
DB_NAME=nome_da_empresa
DB_PORT=5432

JWT_SECRET=seu_jwt_secret_aqui
JWT_REFRESH_SECRET=seu_jwt_refresh_secret_aqui

REDIS_URI=redis://:sua_senha_aqui@127.0.0.1:5000
REDIS_OPT_LIMITER_MAX=1
REGIS_OPT_LIMITER_DURATION=3000

USER_LIMIT=9999
CONNECTIONS_LIMIT=9999
CLOSED_SEND_BY_ME=true

GERENCIANET_SANDBOX=false
GERENCIANET_CLIENT_ID=sua-id
GERENCIANET_CLIENT_SECRET=sua_chave_secreta
GERENCIANET_PIX_CERT=nome_do_certificado
GERENCIANET_PIX_KEY=chave_pix_gerencianet
END
10.2 Instalar dependências do backend
bash
cd /home/deploy/nome_da_empresa/backend
npm install --legacy-peer-deps
10.3 Compilar backend
bash
npm run build
# Ou se tiver erro: npx tsc --build
10.4 Executar migrations e seeds
bash
# Migrations
npx sequelize db:migrate

# Seeds
npx sequelize db:seed:all
10.5 Iniciar backend com PM2
bash
pm2 start dist/server.js --name nome_da_empresa-backend
pm2 save
PARTE 11: CONFIGURAR E COMPILAR FRONTEND
11.1 Criar arquivo .env do frontend
bash
cd /home/deploy/nome_da_empresa/frontend

# Criar .env
cat > .env << 'END'
REACT_APP_BACKEND_URL=https://api.seusite.com
REACT_APP_HOURS_CLOSE_TICKETS_AUTO=24
END
11.2 Criar server.js para o frontend
bash
cat > server.js << 'END'
//simple express server to run frontend production build;
const express = require("express");
const path = require("path");
const app = express();
app.use(express.static(path.join(__dirname, "build")));
app.get("/*", function (req, res) {
	res.sendFile(path.join(__dirname, "build", "index.html"));
});
app.listen(3001);
END
11.3 Instalar dependências do frontend
bash
cd /home/deploy/nome_da_empresa/frontend
npm install --legacy-peer-deps
11.4 Compilar frontend
bash
npm run build
# Isso cria a pasta "build"
11.5 Iniciar frontend com PM2
bash
pm2 start server.js --name nome_da_empresa-frontend
pm2 save
11.6 Verificar processos
bash
pm2 list
pm2 status
PARTE 12: CONFIGURAR NGINX
12.1 Configurar backend
bash
sudo nano /etc/nginx/sites-available/nome_da_empresa-backend
Conteúdo:

nginx
server {
  server_name api.seusite.com;
  
  location / {
    proxy_pass http://127.0.0.1:4000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}
12.2 Configurar frontend
bash
sudo nano /etc/nginx/sites-available/nome_da_empresa-frontend
Conteúdo:

nginx
server {
  server_name app.seusite.com;
  
  location / {
    proxy_pass http://127.0.0.1:3001;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}
12.3 Ativar configurações
bash
# Criar links simbólicos
sudo ln -s /etc/nginx/sites-available/nome_da_empresa-backend /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/nome_da_empresa-frontend /etc/nginx/sites-enabled/

# Testar configuração
sudo nginx -t

# Reiniciar Nginx
sudo systemctl restart nginx
PARTE 13: CONFIGURAR SSL
13.1 Gerar certificados
bash
# Substitua pelos seus domínios e email
sudo certbot --nginx \
  --non-interactive \
  --agree-tos \
  --email seu@email.com \
  --domains api.seusite.com,app.seusite.com
13.2 Verificar renovação automática
bash
sudo certbot renew --dry-run
PARTE 14: CONFIGURAR TIMEZONE
bash
sudo timedatectl set-timezone America/Sao_Paulo
timedatectl
PARTE 15: VERIFICAÇÃO FINAL
15.1 Verificar serviços
bash
# Serviços
systemctl status postgresql
systemctl status docker
systemctl status nginx

# PM2
pm2 list

# Docker
docker ps

# PostgreSQL
sudo -u postgres psql -c "\l"
15.2 Testar URLs
bash
# Testar backend
curl -k https://api.seusite.com

# Testar frontend
curl -k https://app.seusite.com
15.3 Verificar logs se necessário
bash
# Logs do backend
pm2 logs nome_da_empresa-backend

# Logs do Nginx
sudo tail -f /var/log/nginx/error.log
📋 COMANDOS ÚTEIS PARA MANUTENÇÃO
Atualizar sistema
bash
sudo apt update && sudo apt upgrade -y
Atualizar aplicação
bash
# Como usuário deploy
sudo su - deploy
cd /home/deploy/nome_da_empresa

# Parar processos
pm2 stop nome_da_empresa-backend nome_da_empresa-frontend

# Atualizar código
git pull

# Backend
cd backend
npm install --legacy-peer-deps
npm run build
npx sequelize db:migrate

# Frontend
cd ../frontend
npm install --legacy-peer-deps
npm run build

# Reiniciar
pm2 restart nome_da_empresa-backend nome_da_empresa-frontend
pm2 save
Reiniciar tudo
bash
sudo systemctl restart postgresql
sudo systemctl restart docker
sudo systemctl restart nginx
pm2 restart all
Ver logs em tempo real
bash
pm2 logs
journalctl -u nginx -f
✅ PRONTO!
Seu sistema está instalado e funcionando!

Acesse:

Frontend: https://app.seusite.com

Backend: https://api.seusite.com

Qualquer problema, os logs vão ajudar a identificar. 🚀
