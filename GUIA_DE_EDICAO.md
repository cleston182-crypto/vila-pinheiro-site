# Guia de Edição Manual - Vila Pinheiro

Este guia foi criado para que você tenha total autonomia para trocar imagens e adicionar vídeos ao seu site, mesmo sem conhecimento técnico avançado.

## 1. Como Trocar Imagens Manualmente

O site busca as imagens na pasta `client/public`. Para trocar qualquer foto, siga estes passos simples:

### Passo 1: Preparar a Imagem
1.  Escolha a foto que deseja usar.
2.  Renomeie o arquivo para um nome simples, sem acentos ou espaços (ex: `cabana-frente.jpg` em vez de `Foto da Cabana Frente.jpg`).
3.  Recomendamos converter para o formato `.webp` ou `.jpg` para o site carregar rápido (existem sites gratuitos como "Convertio" que fazem isso).

### Passo 2: Colocar a Imagem no Site
1.  Abra a pasta do projeto.
2.  Navegue até `client/public/images/`.
3.  Cole sua nova imagem lá.

### Passo 3: Atualizar o Código
1.  Abra o arquivo `client/src/lib/images.ts` (pode usar o Bloco de Notas ou um editor de código como VS Code).
2.  Procure a linha que tem a imagem que você quer trocar.
    *   *Exemplo:* Se quiser trocar a foto principal da Home, procure por `hero:`.
3.  Mude o nome do arquivo para o nome da sua nova foto.
    *   *Antes:* `hero: "/images/hero-bg.jpg",`
    *   *Depois:* `hero: "/images/minha-nova-foto.jpg",`
4.  Salve o arquivo. Pronto! A imagem será atualizada automaticamente.

---

## 2. Como Adicionar um Vídeo em Loop na Home

Para colocar um vídeo de fundo na página inicial (Home), siga este tutorial.

### Passo 1: Preparar o Vídeo
1.  O vídeo deve ser **curto** (5 a 10 segundos) e **leve** (menos de 5MB é o ideal) para não travar o celular dos clientes.
2.  O formato deve ser `.mp4`.
3.  Nomeie o arquivo como `hero-video.mp4`.
4.  Coloque este arquivo na pasta `client/public/videos/` (se a pasta `videos` não existir dentro de `public`, crie ela).

### Passo 2: Alterar o Código da Home
1.  Abra o arquivo `client/src/pages/Home.tsx`.
2.  Procure pela seção "Hero Section" (logo no começo do arquivo).
3.  Substitua o código da imagem pelo código do vídeo.

**Código para Copiar e Colar:**

Substitua este trecho (que mostra a imagem):
```tsx
<div className="absolute inset-0 z-0">
  <img 
    src={images.home.hero} 
    alt="Vila Pinheiro Santa Teresa" 
    className="w-full h-full object-cover"
  />
  <div className="absolute inset-0 bg-black/30" />
</div>
```

Por este trecho (que mostra o vídeo):
```tsx
<div className="absolute inset-0 z-0">
  <video 
    autoPlay 
    loop 
    muted 
    playsInline 
    className="w-full h-full object-cover"
  >
    <source src="/videos/hero-video.mp4" type="video/mp4" />
    {/* Fallback para imagem caso o vídeo não carregue */}
    <img 
      src={images.home.hero} 
      alt="Vila Pinheiro Santa Teresa" 
      className="w-full h-full object-cover"
    />
  </video>
  <div className="absolute inset-0 bg-black/30" />
</div>
```

### Dica Importante sobre Vídeos
Vídeos pesados fazem o site demorar para abrir e o Google "punir" sua posição nas buscas. Use vídeos apenas se forem bem otimizados!

---

## 3. Estrutura de Pastas Importantes

*   `client/public/`: Onde ficam todos os arquivos de mídia (fotos, vídeos, ícones).
*   `client/src/pages/`: Onde ficam os textos e a estrutura de cada página (Home, Sobre, Cabanas).
*   `client/src/lib/images.ts`: O "mapa" que diz qual imagem vai em qual lugar.

Se tiver dúvidas, pode consultar este guia a qualquer momento!
