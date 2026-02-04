# Integração de Imagens Dinâmicas - Vila Pinheiro

Este documento descreve as mudanças implementadas para integrar imagens dinâmicas do banco de dados ao site.

## O que foi alterado?

### 1. Novos Hooks (`client/src/hooks/useImages.ts`)

Foram criados hooks customizados para buscar imagens do banco de dados:

```typescript
// Buscar imagens por categoria
useImagesByCategory(category, subcategory?)

// Buscar todas as imagens
useAllImages()

// Utilitários para extrair URLs
getFirstImageUrl(images, fallback)
getImageGallery(images, fallback)
```

**Exemplo de uso:**

```typescript
const homeImagesQuery = useImagesByCategory("home");
const heroImage = getFirstImageUrl(homeImagesQuery.data, images.home.hero);
```

### 2. Novo Componente (`client/src/components/DynamicImage.tsx`)

Componente que exibe imagens com:
- Fallback automático para URLs estáticas
- Tratamento de erros
- Loading skeleton
- Suporte a lazy loading

**Exemplo de uso:**

```tsx
<DynamicImage
  src={heroImage}
  fallback={images.home.hero}
  alt="Vila Pinheiro"
  className="w-full h-full object-cover"
  isLoading={isLoading}
/>
```

### 3. Páginas Atualizadas

#### **Home.tsx**
- Busca imagens da categoria "home"
- Exibe hero, intro e experience com imagens dinâmicas
- Fallback para URLs estáticas se nenhuma imagem for encontrada

#### **Cabanas.tsx**
- Busca imagens de cada cabana (ipe, manaca, jasmim, floresta)
- Atualiza cards com imagens do banco de dados
- Mantém compatibilidade com dados estáticos

#### **CabanaDetail.tsx**
- Busca galeria completa da cabana
- Exibe header image e grid de fotos
- Suporta múltiplas imagens por cabana

### 4. Componente de Edição de Conteúdo (`client/src/components/ContentEditor.tsx`)

Novo componente para editar textos do site sem mexer no código:

```tsx
<ContentEditor
  items={[
    { id: "titulo", label: "Título", value: "Bem-vindo", type: "text" },
    { id: "descricao", label: "Descrição", value: "...", type: "textarea" }
  ]}
  onSave={handleSave}
  title="Editar Conteúdo"
/>
```

## Como usar no Admin?

### 1. Fazer Upload de Imagens

1. Acesse `/admin`
2. Faça login com sua conta
3. Selecione uma imagem
4. Escolha a categoria:
   - **Home**: Imagens da página inicial
   - **Cabanas**: Imagens das cabanas (selecione a subcategoria)
   - **Sobre**: Imagens da página sobre
   - **Depoimentos**: Imagens de depoimentos

5. Clique em "Enviar Imagem"

### 2. Organizar Imagens

As imagens são ordenadas por data de upload (mais recentes primeiro). A primeira imagem de cada categoria será usada como imagem principal.

### 3. Deletar Imagens

Passe o mouse sobre a imagem e clique no ícone de lixeira.

## Como as Imagens são Selecionadas?

O sistema usa a seguinte lógica:

1. **Home**: Filtra por nome do arquivo (busca por "hero", "intro", "experience")
2. **Cabanas**: Busca por categoria + subcategoria
3. **Fallback**: Se nenhuma imagem for encontrada, usa as URLs estáticas

## Performance

### Cache

- Imagens são cacheadas por **5 minutos**
- Garbage collection após **10 minutos**
- Reduz requisições ao servidor

### Otimizações

- Lazy loading de imagens
- Skeleton loading enquanto carrega
- Tratamento de erros com fallback automático

## Estrutura do Banco de Dados

```sql
CREATE TABLE images (
  id INT PRIMARY KEY AUTO_INCREMENT,
  fileKey VARCHAR(512) NOT NULL,
  url TEXT NOT NULL,
  filename VARCHAR(255) NOT NULL,
  mimeType VARCHAR(100) NOT NULL,
  fileSize INT NOT NULL,
  category VARCHAR(50) NOT NULL,
  subcategory VARCHAR(50),
  uploadedBy INT NOT NULL,
  createdAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updatedAt TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

## Endpoints tRPC

### `images.upload` (Protegido)

Upload de nova imagem:

```typescript
trpc.images.upload.useMutation({
  filename: "foto.jpg",
  mimeType: "image/jpeg",
  fileSize: 1024000,
  category: "cabanas",
  subcategory: "ipe",
  base64Data: "..." // dados em base64
})
```

### `images.list` (Público)

Listar todas as imagens:

```typescript
trpc.images.list.useQuery()
```

### `images.listByCategory` (Público)

Listar imagens por categoria:

```typescript
trpc.images.listByCategory.useQuery({
  category: "cabanas",
  subcategory: "ipe"
})
```

### `images.delete` (Protegido)

Deletar imagem:

```typescript
trpc.images.delete.useMutation({
  id: 123
})
```

## Próximos Passos Sugeridos

### 1. Edição de Textos no Admin

Adicionar componente `ContentEditor` ao Admin para editar:
- Títulos das cabanas
- Descrições
- Textos de depoimentos
- Informações de contato

### 2. Ordenação de Imagens

Implementar drag-and-drop para reordenar imagens:

```typescript
// Adicionar campo de ordem ao banco
ALTER TABLE images ADD COLUMN `order` INT DEFAULT 0;

// Atualizar endpoint para ordenar por este campo
```

### 3. Filtros Avançados

- Filtrar por data
- Buscar por nome
- Visualizar estatísticas de uploads

### 4. Compressão de Imagens

Implementar compressão automática ao fazer upload:

```typescript
// Usar biblioteca como sharp ou imagemin
const compressed = await sharp(buffer)
  .resize(1920, 1080, { fit: 'inside' })
  .webp({ quality: 80 })
  .toBuffer();
```

### 5. Múltiplas Resoluções

Gerar versões otimizadas das imagens:
- Thumbnail (200x200)
- Medium (800x600)
- Large (1920x1080)

## Troubleshooting

### Imagens não aparecem

1. Verificar se o upload foi bem-sucedido no Admin
2. Verificar se a categoria está correta
3. Verificar logs do navegador (F12 > Console)
4. Verificar se o S3 está acessível

### Erro ao fazer upload

1. Verificar tamanho da imagem (máximo recomendado: 5MB)
2. Verificar formato (JPG, PNG, WebP, etc)
3. Verificar se a conta tem permissão de admin
4. Verificar se as credenciais do S3 estão corretas

### Performance lenta

1. Reduzir tamanho das imagens
2. Aumentar cache time
3. Implementar CDN (Cloudflare)
4. Otimizar queries do banco de dados

## Arquivos Modificados

- `client/src/pages/Home.tsx` - Integração de imagens dinâmicas
- `client/src/pages/Cabanas.tsx` - Integração de imagens dinâmicas
- `client/src/pages/CabanaDetail.tsx` - Integração de imagens dinâmicas

## Arquivos Criados

- `client/src/hooks/useImages.ts` - Hooks para buscar imagens
- `client/src/components/DynamicImage.tsx` - Componente de imagem dinâmica
- `client/src/components/ContentEditor.tsx` - Editor de conteúdo
- `DEPLOYMENT_GUIDE.md` - Guia de deploy
- `README_INTEGRACAO.md` - Este arquivo

## Compatibilidade

- ✅ React 19+
- ✅ TypeScript 5.9+
- ✅ Tailwind CSS 4+
- ✅ tRPC 11+
- ✅ Drizzle ORM 0.44+

## Suporte

Para dúvidas sobre a integração, consulte:

- [Documentação tRPC](https://trpc.io)
- [Documentação React Query](https://tanstack.com/query)
- [Documentação Drizzle](https://orm.drizzle.team)

---

**Versão:** 1.0.0  
**Data:** Fevereiro 2026
