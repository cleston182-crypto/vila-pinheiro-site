# Deploy Passo a Passo - Vila Pinheiro

Guia prático para colocar o site no ar em 30 minutos.

## 🎯 Objetivo Final

Ter o site rodando em um domínio público com banco de dados e S3 configurados.

## ⏱️ Tempo Estimado

- Setup: 5 minutos
- Deploy: 10 minutos
- Testes: 15 minutos
- **Total: 30 minutos**

## 📋 Pré-requisitos

Antes de começar, tenha em mãos:

- [ ] Conta GitHub (para conectar repositório)
- [ ] Conta Vercel ou Railway (hospedagem)
- [ ] Conta AWS com S3 bucket criado
- [ ] Banco de dados MySQL/TiDB criado
- [ ] Domínio registrado (opcional)

## 🚀 Passo 1: Preparar o Repositório

### 1.1 Fazer Push para GitHub

```bash
cd vila-pinheiro-site
git init
git add .
git commit -m "Initial commit: Vila Pinheiro full-stack"
git branch -M main
git remote add origin https://github.com/seu-usuario/vila-pinheiro.git
git push -u origin main
```

### 1.2 Criar arquivo `.env.production`

Na raiz do projeto, criar:

```env
# Banco de Dados
DATABASE_URL=mysql://user:password@host:3306/database

# AWS S3
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...
AWS_S3_BUCKET=vila-pinheiro-prod

# Manus OAuth
MANUS_CLIENT_ID=...
MANUS_CLIENT_SECRET=...
MANUS_REDIRECT_URI=https://seu-dominio.com/auth/callback

# Segurança
SESSION_SECRET=gere_uma_string_aleatoria_segura
NODE_ENV=production

# Proprietário
OWNER_OPEN_ID=seu_open_id

# URL Pública
PUBLIC_URL=https://seu-dominio.com
```

**NÃO fazer commit deste arquivo!** Adicionar ao `.gitignore`:

```bash
echo ".env.production" >> .gitignore
git add .gitignore
git commit -m "Add .env.production to gitignore"
git push
```

## 🌐 Passo 2: Escolher Hospedagem

### Opção A: Vercel (Recomendado)

#### 2A.1 Conectar Vercel

1. Ir para [vercel.com](https://vercel.com)
2. Fazer login com GitHub
3. Clicar em "New Project"
4. Selecionar repositório `vila-pinheiro`
5. Clicar em "Import"

#### 2A.2 Configurar Variáveis de Ambiente

1. Em "Environment Variables", adicionar:
   - `DATABASE_URL`
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `AWS_S3_BUCKET`
   - `MANUS_CLIENT_ID`
   - `MANUS_CLIENT_SECRET`
   - `MANUS_REDIRECT_URI`
   - `SESSION_SECRET`
   - `OWNER_OPEN_ID`
   - `PUBLIC_URL`

2. Clicar em "Deploy"

#### 2A.3 Apontar Domínio

1. Em "Settings" > "Domains"
2. Adicionar seu domínio
3. Seguir instruções de DNS

### Opção B: Railway

#### 2B.1 Conectar Railway

1. Ir para [railway.app](https://railway.app)
2. Fazer login com GitHub
3. Clicar em "New Project"
4. Selecionar "Deploy from GitHub repo"
5. Selecionar repositório

#### 2B.2 Configurar Banco de Dados

1. Em "New", selecionar "MySQL"
2. Conectar ao projeto
3. Copiar `DATABASE_URL`

#### 2B.3 Adicionar Variáveis de Ambiente

1. Em "Variables", adicionar todas as variáveis
2. Clicar em "Deploy"

### Opção C: Self-Hosted (DigitalOcean/Linode)

#### 2C.1 Criar Droplet

```bash
# SSH para o servidor
ssh root@seu-ip

# Atualizar sistema
apt update && apt upgrade -y

# Instalar Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
apt install -y nodejs

# Instalar pnpm
npm install -g pnpm

# Clonar repositório
git clone https://github.com/seu-usuario/vila-pinheiro.git
cd vila-pinheiro
```

#### 2C.2 Configurar Variáveis de Ambiente

```bash
nano .env.production
# Adicionar todas as variáveis
```

#### 2C.3 Build e Start

```bash
pnpm install
pnpm build
npm install -g pm2
pm2 start dist/index.js --name "vila-pinheiro"
pm2 save
```

## 🗄️ Passo 3: Configurar Banco de Dados

### 3.1 Criar Banco de Dados

**MySQL Remoto:**

```bash
# Conectar ao servidor MySQL
mysql -u admin -p -h seu-host.com

# Criar banco
CREATE DATABASE vila_pinheiro;
CREATE USER 'vila_user'@'%' IDENTIFIED BY 'senha_segura_123';
GRANT ALL PRIVILEGES ON vila_pinheiro.* TO 'vila_user'@'%';
FLUSH PRIVILEGES;
```

**TiDB Cloud:**

1. Ir para [tidb.cloud](https://tidb.cloud)
2. Criar cluster
3. Copiar connection string

### 3.2 Executar Migrações

```bash
# Localmente
DATABASE_URL="mysql://user:pass@host/db" pnpm db:push

# Ou no servidor
ssh seu-usuario@seu-servidor
cd vila-pinheiro
DATABASE_URL="..." pnpm db:push
```

## 🪣 Passo 4: Configurar AWS S3

### 4.1 Criar Bucket S3

1. Ir para [AWS Console](https://console.aws.amazon.com)
2. Ir para S3
3. Clicar em "Create bucket"
4. Nome: `vila-pinheiro-prod`
5. Região: `us-east-1` (ou sua região)
6. Desabilitar "Block all public access"
7. Criar bucket

### 4.2 Criar Credenciais IAM

1. Ir para IAM
2. Clicar em "Users"
3. Clicar em "Create user"
4. Nome: `vila-pinheiro-app`
5. Clicar em "Create access key"
6. Copiar `Access Key ID` e `Secret Access Key`

### 4.3 Adicionar Permissões

1. Ir para "Policies"
2. Clicar em "Create policy"
3. Adicionar JSON:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::vila-pinheiro-prod/*"
    }
  ]
}
```

4. Clicar em "Create policy"
5. Ir para usuário e adicionar policy

## 🔐 Passo 5: Configurar Manus OAuth

### 5.1 Registrar Aplicação

1. Ir para [manus.im](https://manus.im)
2. Fazer login ou criar conta
3. Ir para "Applications"
4. Clicar em "New Application"
5. Preencher:
   - **Name:** Vila Pinheiro
   - **Redirect URI:** `https://seu-dominio.com/auth/callback`
6. Copiar `Client ID` e `Client Secret`

### 5.2 Adicionar ao `.env.production`

```env
MANUS_CLIENT_ID=seu_client_id
MANUS_CLIENT_SECRET=seu_client_secret
MANUS_REDIRECT_URI=https://seu-dominio.com/auth/callback
```

## ✅ Passo 6: Testar Deploy

### 6.1 Verificar Build

```bash
# Localmente
pnpm build
pnpm start

# Acessar http://localhost:3000
```

### 6.2 Testar Funcionalidades

- [ ] Home page carrega
- [ ] Imagens aparecem
- [ ] Admin page carrega
- [ ] Upload de imagem funciona
- [ ] Formulário de reserva funciona
- [ ] WhatsApp link funciona

### 6.3 Verificar Logs

**Vercel:**
```bash
vercel logs
```

**Railway:**
```bash
railway logs
```

**Self-hosted:**
```bash
pm2 logs vila-pinheiro
```

## 🌍 Passo 7: Configurar Domínio

### 7.1 Apontar DNS

**Para Vercel:**

1. Ir para registrador de domínio
2. Adicionar CNAME:
   - Host: `seu-dominio.com`
   - Value: `cname.vercel-dns.com`

**Para Railway:**

1. Ir para Railway > Settings > Domains
2. Adicionar domínio
3. Seguir instruções

**Para Self-hosted:**

1. Apontar A record para IP do servidor
2. Configurar Nginx/Apache

### 7.2 Ativar SSL/HTTPS

**Vercel:** Automático com Let's Encrypt

**Railway:** Automático

**Self-hosted:**

```bash
# Instalar Certbot
apt install certbot python3-certbot-nginx

# Gerar certificado
certbot certonly --standalone -d seu-dominio.com

# Configurar Nginx
# Adicionar certificado ao nginx.conf
```

## 🎉 Passo 8: Verificar Tudo

Acessar `https://seu-dominio.com` e verificar:

- [ ] Site carrega corretamente
- [ ] Imagens aparecem
- [ ] Responsividade funciona
- [ ] Admin page acessível
- [ ] Formulário funciona
- [ ] Sem erros no console

## 🚨 Troubleshooting Rápido

### Erro: "Database connection failed"

```bash
# Verificar DATABASE_URL
echo $DATABASE_URL

# Testar conexão
mysql -u user -p -h host database
```

### Erro: "AWS S3 access denied"

```bash
# Verificar credenciais
aws s3 ls --profile default

# Verificar bucket
aws s3 ls s3://vila-pinheiro-prod/
```

### Site não carrega

```bash
# Verificar logs
vercel logs
# ou
railway logs
# ou
pm2 logs vila-pinheiro

# Verificar variáveis
echo $DATABASE_URL
echo $AWS_ACCESS_KEY_ID
```

### Imagens não carregam

1. Verificar se upload foi bem-sucedido
2. Verificar URL da imagem no S3
3. Verificar permissões do bucket

## 📞 Suporte

Se tiver problemas:

1. Consultar logs (ver Troubleshooting)
2. Verificar documentação (DEPLOYMENT_GUIDE.md)
3. Contatar suporte da hospedagem

## ✨ Próximos Passos

Após o deploy:

- [ ] Configurar backups automáticos
- [ ] Implementar monitoramento (Sentry)
- [ ] Adicionar analytics (Google Analytics)
- [ ] Configurar CI/CD automático
- [ ] Testar performance (PageSpeed Insights)

---

**Parabéns! Seu site está no ar!** 🎉

Para mais detalhes, consulte [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md)
