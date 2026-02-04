# Quick Start - Vila Pinheiro Site

Guia rápido para começar a usar o site.

## 🚀 Início Rápido

### 1. Instalação

```bash
# Clonar/extrair o projeto
cd vila-pinheiro-site

# Instalar dependências
pnpm install

# Verificar tipos
pnpm check
```

### 2. Configuração Local

Criar arquivo `.env.local`:

```env
# Banco de Dados (local)
DATABASE_URL=mysql://root:password@localhost:3306/vila_pinheiro

# AWS S3
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=sua_chave
AWS_SECRET_ACCESS_KEY=sua_secreta
AWS_S3_BUCKET=seu-bucket

# Manus OAuth (obter em manus.im)
MANUS_CLIENT_ID=seu_client_id
MANUS_CLIENT_SECRET=seu_client_secret
MANUS_REDIRECT_URI=http://localhost:3000/auth/callback

# Segurança
SESSION_SECRET=dev_secret_key_123456789

# Proprietário
OWNER_OPEN_ID=seu_open_id
```

### 3. Banco de Dados

```bash
# Criar banco de dados
mysql -u root -p -e "CREATE DATABASE vila_pinheiro;"

# Executar migrações
pnpm db:push
```

### 4. Executar Localmente

```bash
# Desenvolvimento
pnpm dev

# Acesse http://localhost:5173
```

## 📁 Estrutura do Projeto

```
vila-pinheiro-site/
├── client/                    # Frontend React
│   └── src/
│       ├── pages/            # Páginas do site
│       ├── components/       # Componentes reutilizáveis
│       ├── hooks/            # Hooks customizados (incluindo useImages)
│       ├── lib/              # Utilitários
│       └── index.css         # Estilos globais
├── server/                    # Backend Express
│   ├── routers.ts            # Endpoints tRPC
│   ├── db.ts                 # Funções de banco de dados
│   └── storage.ts            # Upload para S3
├── drizzle/                   # Migrações do banco
│   └── schema.ts             # Schema das tabelas
├── shared/                    # Código compartilhado
└── package.json              # Dependências
```

## 🎨 Páginas Principais

| Página | URL | Descrição |
|--------|-----|-----------|
| Home | `/` | Página inicial com hero section |
| Cabanas | `/cabanas` | Lista de todas as cabanas |
| Detalhes | `/cabanas/:id` | Detalhes de uma cabana |
| Reservas | `/reservas` | Formulário de reserva via WhatsApp |
| Sobre | `/sobre` | Sobre a Vila Pinheiro |
| Regras | `/regras` | Regras e políticas |
| Admin | `/admin` | Painel administrativo |

## 🖼️ Gerenciar Imagens

### Fazer Upload

1. Acesse `/admin` (faça login)
2. Selecione uma imagem
3. Escolha a categoria:
   - **home** - Página inicial
   - **cabanas** - Fotos das cabanas (selecione qual)
   - **about** - Página sobre
   - **testimonials** - Depoimentos
4. Clique em "Enviar Imagem"

### Usar Imagens no Código

```typescript
// Hook para buscar imagens
import { useImagesByCategory, getFirstImageUrl } from "@/hooks/useImages";

const imagesQuery = useImagesByCategory("cabanas", "ipe");
const mainImage = getFirstImageUrl(imagesQuery.data, fallbackUrl);

// Componente para exibir
import { DynamicImage } from "@/components/DynamicImage";

<DynamicImage
  src={mainImage}
  fallback={fallbackUrl}
  alt="Cabana Ipê"
  isLoading={imagesQuery.isLoading}
/>
```

## 🔧 Comandos Úteis

```bash
# Desenvolvimento
pnpm dev              # Inicia servidor de desenvolvimento

# Build
pnpm build            # Build para produção
pnpm start            # Executa build de produção

# Qualidade
pnpm check            # Verifica tipos TypeScript
pnpm format           # Formata código com Prettier
pnpm test             # Executa testes

# Banco de Dados
pnpm db:push          # Executa migrações
```

## 🔐 Autenticação

O site usa **Manus OAuth** para autenticação:

1. Criar conta em [manus.im](https://manus.im)
2. Criar aplicação OAuth
3. Obter `CLIENT_ID` e `CLIENT_SECRET`
4. Adicionar ao `.env.local`
5. Usuários podem fazer login via `/auth/login`

## 📊 Admin Panel

O painel administrativo permite:

- ✅ Upload de imagens
- ✅ Gerenciar galeria
- ✅ Deletar imagens
- ✅ Visualizar estatísticas

**Acesso:** `/admin` (requer autenticação)

## 🚢 Deploy

Veja [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md) para instruções completas.

### Deploy Rápido (Vercel)

```bash
# Instalar Vercel CLI
npm i -g vercel

# Deploy
vercel --prod
```

### Deploy com Docker

```bash
docker build -t vila-pinheiro .
docker run -p 3000:3000 \
  -e DATABASE_URL="..." \
  -e AWS_ACCESS_KEY_ID="..." \
  -e AWS_SECRET_ACCESS_KEY="..." \
  -e AWS_S3_BUCKET="..." \
  vila-pinheiro
```

## 🐛 Troubleshooting

### Erro: "Database connection failed"

```bash
# Verificar conexão MySQL
mysql -u root -p -h localhost -e "SELECT 1;"

# Verificar DATABASE_URL
echo $DATABASE_URL
```

### Erro: "AWS S3 access denied"

```bash
# Verificar credenciais S3
aws s3 ls --profile default
```

### Porta 3000 já em uso

```bash
# Usar porta diferente
PORT=3001 pnpm dev
```

### Imagens não carregam

1. Verificar se upload foi bem-sucedido no Admin
2. Verificar se S3 está acessível
3. Verificar logs do navegador (F12)

## 📚 Documentação

- [README_INTEGRACAO.md](./README_INTEGRACAO.md) - Detalhes da integração de imagens
- [DEPLOYMENT_GUIDE.md](./DEPLOYMENT_GUIDE.md) - Guia completo de deploy
- [Documentação tRPC](https://trpc.io)
- [Documentação React](https://react.dev)
- [Documentação Tailwind](https://tailwindcss.com)

## 🆘 Suporte

Para dúvidas ou problemas:

1. Verificar documentação acima
2. Consultar logs do projeto
3. Verificar console do navegador (F12)
4. Contatar suporte Manus

## ✅ Checklist de Deploy

- [ ] Variáveis de ambiente configuradas
- [ ] Banco de dados criado e migrado
- [ ] S3 bucket criado e acessível
- [ ] Build local testado (`pnpm build`)
- [ ] Testes passando (`pnpm test`)
- [ ] Tipos verificados (`pnpm check`)
- [ ] Domínio apontado para hospedagem
- [ ] SSL/HTTPS ativado
- [ ] Backups configurados
- [ ] Monitoramento ativado

---

**Pronto para começar?** Execute `pnpm dev` e acesse `http://localhost:5173`! 🎉
