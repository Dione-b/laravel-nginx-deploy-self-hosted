# 1ª Etapa 
#### Configurar Projeto Github

```bash
git init
git remote add origin <link-do-repositório-git>
git add .
git commit -m "first deploy"
git branch -M main
git push -u origin main
```

# 2ª Etapa 
#### Configurar Sub-Domínio na Cloudflare

```bash
Tipo - A
Name - <sub-domínio> (admin) -> admin.meudominio.com
Content - <ip-servidor> (181.158.15.89) -> fictício
Proxy status - DNS only -> (irei configurar o ssl com o certbot)
TTL - Auto
```

# 3ª Etapa 
#### Acessar servidor SSH

```bash
ssh -i ~/.ssh/id_ed25519 root@meudominio.com
```

> é necessário configurar o ssh-client na máquina antes

# 4ª Etapa 
#### Configurar diretório de produção

```bash
cd /var/www/html

sudo mkdir admin.meudominio.com

sudo chmod 775 admin.meudominio.com
```

# 5ª Etapa 
#### Configurar Sub-Domínio no servidor Nginx

```bash
sudo cp /etc/nginx/sites-available/api.meudominio.com /etc/nginx/sites-available/admin.meudominio.com

OU

cd /etc/nginx/sites-available/

sudo nano admin.meudominio.com
```
> caso você esteja implantando pela primeira vez, basta criar um arquivo sem extensão dentro da pasta `/etc/nginx/sites-available/` e inserir o script de configuração do nginx para a stack usada.

#### Depois colar script dentro de admin.meudominio.com:

```bash
	server {
	    listen 80;
	    server_name admin.meudominio.com;
	    root /var/www/admin.meudominio.com/public;

	    add_header X-Frame-Options "SAMEORIGIN";
	    add_header X-XSS-Protection "1; mode=block";
	    add_header X-Content-Type-Options "nosniff";

	    index index.html index.htm index.php;

	    charset utf-8;

	    location / {
	        try_files $uri $uri/ /index.php?$query_string;
	    }

	    location = /favicon.ico { access_log off; log_not_found off; }
	    location = /robots.txt  { access_log off; log_not_found off; }

	    error_page 404 /index.php;

	    location ~ \.php$ {
	        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
	        fastcgi_index index.php;
	        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
	        include fastcgi_params;
	    }

	    location ~ /\.(?!well-known).* {
	        deny all;
	    }
	}
```
#### Aqui é criado um atalho de configuração para o nginx habilitar o seu site.

```bash
sudo ln -s /etc/nginx/sites-available/admin.meudominio.com /etc/nginx/sites-enabled/

sudo systemctl restart nginx
```
# 6ª Etapa 

#### Configurar arquivos para produção. Primeiro clone o repositório no diretório de produção:

```bash
cd /var/www/html/admin.meudominio.com

git clone <link-do-repositório> .
```
#### Execute as permissões para o projeto laravel:

```bash
sudo chown -R www-data:www-data /var/www/html/admin.meudominio.com/storage

sudo chmod -R 775 /var/www/html/admin.meudominio.com/storage

sudo chown -R www-data.www-data /var/www/html/admin.meudominio.com/bootstrap/cache
```

#### Execute os comandos para deploy:

```bash
cp .env.example .env

php artisan key:generate

php artisan config:cache

php artisan migrate
```

# 7ª Etapa 
#### configurar certificado SSL:

```bash
sudo certbot --nginx -d admin.meudominio.com

sudo systemctl restart nginx
```
