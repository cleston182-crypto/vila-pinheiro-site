# Guia de Deploy - Vila Pinheiro Site

Este documento fornece instruções passo a passo para colocar o site da Vila Pinheiro em produção.

## Índice

1. [Pré-requisitos](#pré-requisitos)
2. [Preparação do Projeto](#preparação-do-projeto)
3. [Configuração do Banco de Dados](#configuração-do-banco-de-dados)
4. [Variáveis de Ambiente](#variáveis-de-ambiente)
5. [Build e Deploy](#build-e-deploy)
6. [Opções de Hospedagem](#opções-de-hospedagem)
7. [Troubleshooting](#troubleshooting)

## Pré-requisitos

Antes de começar, certifique-se de ter:

- **Node.js** 18+ instalado
- **pnpm** como gerenciador de pacotes
- Uma conta em um serviço de hospedagem (Vercel, Netlify, Railway, etc.)
- Um banco de dados MySQL/TiDB em produção
- Uma conta AWS S3 para armazenamento de imagens
- Um domínio registrado (opcional, mas recomendado)

## Preparação do Projeto

### 1. Instalar Dependências

```bash
cd vila-pinheiro-site
pnpm install
```

### 2. Verificar Tipos TypeScript

```bash
pnpm check
```

### 3. Executar Testes

```bash
pnpm test
```

### 4. Build Local

```bash
pnpm build
```

Isso irá:
- Compilar o frontend com Vite
- Compilar o backend com esbuild
- Gerar arquivos otimizados em `dist/`

## Configuração do Banco de Dados

### Opção 1: MySQL Tradicional

1. **Criar banco de dados:**

```sql
CREATE DATABASE vila_pinheiro;
CREATE USER 'vila_user'@'localhost' IDENTIFIED BY 'senha_segura';
GRANT ALL PRIVILEGES ON vila_pinheiro.* TO 'vila_user'@'localhost';
FLUSH PRIVILEGES;
```

2. **String de conexão:**

```
DATABASE_URL=mysql://vila_user:senha_segura@localhost:3306/vila_pinheiro
```

### Opção 2: TiDB Cloud (Recomendado para Produção)

1. Criar cluster em [tidb.cloud](https://tidb.cloud)
2. Obter connection string
3. Usar como `DATABASE_URL`

### Executar Migrações

```bash
pnpm db:push
```

Isso irá:
- Gerar migrações Drizzle
- Aplicar schema ao banco de dados
- Criar tabelas necessárias

## Variáveis de Ambiente

Crie um arquivo `.env.production` com as seguintes variáveis:

```env
# Banco de Dados
DATABASE_URL=mysql://user:password@host:3306/database

# AWS S3 (para upload de imagens)
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=sua_chave_aqui
AWS_SECRET_ACCESS_KEY=sua_secreta_aqui
AWS_S3_BUCKET=seu-bucket-vila-pinheiro

# Autenticação (Manus OAuth)
MANUS_CLIENT_ID=seu_client_id
MANUS_CLIENT_SECRET=seu_client_secret
MANUS_REDIRECT_URI=https://seu-dominio.com/auth/callback

# Segurança
SESSION_SECRET=gere_uma_string_aleatoria_segura_aqui
NODE_ENV=production

# Informações do Proprietário (para role de admin)
OWNER_OPEN_ID=seu_open_id_do_manus

# URL Pública
PUBLIC_URL=https://seu-dominio.com
```

### Gerar SESSION_SECRET Seguro

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

## Build e Deploy

### 1. Build Final

```bash
pnpm build
```

Verifique se o arquivo `dist/index.js` foi criado corretamente.

### 2. Testar Localmente

```bash
NODE_ENV=production node dist/index.js
```

Acesse `http://localhost:3000` para verificar.

### 3. Deploy em Diferentes Plataformas

#### **Vercel** (Recomendado para Full-Stack)

```bash
# Instalar CLI
npm i -g vercel

# Deploy
vercel --prod
```

**Configuração necessária em `vercel.json`:**

```json
{
  "buildCommand": "pnpm build",
  "outputDirectory": "dist",
  "env": {
    "DATABASE_URL": "@database_url",
    "AWS_ACCESS_KEY_ID": "@aws_access_key_id",
    "AWS_SECRET_ACCESS_KEY": "@aws_secret_access_key",
    "AWS_S3_BUCKET": "@aws_s3_bucket",
    "MANUS_CLIENT_ID": "@manus_client_id",
    "MANUS_CLIENT_SECRET": "@manus_client_secret",
    "SESSION_SECRET": "@session_secret"
  }
}
```

#### **Railway** (Suporta Node.js bem)

1. Conectar repositório GitHub
2. Adicionar variáveis de ambiente no painel
3. Deploy automático ao fazer push

#### **Docker + Self-Hosted**

**Dockerfile:**

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile

COPY . .

RUN pnpm build

EXPOSE 3000

ENV NODE_ENV=production

CMD ["node", "dist/index.js"]
```

**Build e run:**

```bash
docker build -t vila-pinheiro .
docker run -p 3000:3000 \
  -e DATABASE_URL="mysql://..." \
  -e AWS_ACCESS_KEY_ID="..." \
  -e AWS_SECRET_ACCESS_KEY="..." \
  -e AWS_S3_BUCKET="..." \
  vila-pinheiro
```

## Opções de Hospedagem

| Plataforma | Custo | Facilidade | Escalabilidade | Recomendação |
|-----------|-------|-----------|----------------|--------------|
| **Vercel** | $20-100/mês | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ✅ Melhor para começar |
| **Railway** | $5-50/mês | ⭐⭐⭐⭐ | ⭐⭐⭐ | ✅ Bom custo-benefício |
| **Render** | $7-50/mês | ⭐⭐⭐⭐ | ⭐⭐⭐ | ✅ Alternativa a Railway |
| **DigitalOcean** | $5-100/mês | ⭐⭐⭐ | ⭐⭐⭐⭐ | ✅ Mais controle |
| **AWS EC2** | $5-200/mês | ⭐⭐ | ⭐⭐⭐⭐⭐ | Para escala grande |

## Configuração de Domínio

### Apontar Domínio para Vercel

1. Ir para seu registrador de domínio
2. Adicionar registros DNS:
   - **CNAME**: `seu-dominio.com` → `cname.vercel-dns.com`
   - **TXT**: Verificação de domínio (fornecido por Vercel)

### SSL/HTTPS

- **Vercel**: Automático com Let's Encrypt
- **Railway**: Automático
- **Self-hosted**: Use Nginx + Certbot para Let's Encrypt

## Monitoramento e Manutenção

### Logs

```bash
# Vercel
vercel logs

# Railway
railway logs

# Self-hosted
pm2 logs vila-pinheiro
```

### Backup do Banco de Dados

```bash
# MySQL
mysqldump -u user -p database > backup.sql

# Restaurar
mysql -u user -p database < backup.sql
```

### Atualizar Código

```bash
git pull origin main
pnpm install
pnpm build
# Redeploy automático (Vercel/Railway) ou manual (self-hosted)
```

## Troubleshooting

### Erro: "Database connection failed"

- Verificar `DATABASE_URL`
- Confirmar que o banco está acessível
- Testar conexão: `mysql -u user -p -h host database`

### Erro: "AWS S3 access denied"

- Verificar `AWS_ACCESS_KEY_ID` e `AWS_SECRET_ACCESS_KEY`
- Confirmar que a chave tem permissão para S3
- Verificar se o bucket existe

### Erro: "Build failed"

```bash
# Limpar cache
rm -rf node_modules pnpm-lock.yaml
pnpm install
pnpm build
```

### Site lento

- Ativar compressão Gzip
- Otimizar imagens (usar WebP)
- Implementar CDN (Cloudflare)
- Aumentar recursos do servidor

### Problema com imagens não carregando

- Verificar permissões do S3
- Confirmar que URLs públicas estão corretas
- Testar acesso direto à URL da imagem

## Próximos Passos

1. ✅ Configurar domínio customizado
2. ✅ Implementar SSL/HTTPS
3. ✅ Configurar backups automáticos
4. ✅ Implementar monitoramento (Sentry, DataDog)
5. ✅ Configurar CI/CD automático
6. ✅ Adicionar analytics (Google Analytics, Plausible)

## Suporte

Para dúvidas sobre deploy, consulte:

- [Documentação Vercel](https://vercel.com/docs)
- [Documentação Railway](https://docs.railway.app)
- [Documentação Drizzle](https://orm.drizzle.team)
- [Documentação tRPC](https://trpc.io/docs)

---

**Última atualização:** Fevereiro 2026
