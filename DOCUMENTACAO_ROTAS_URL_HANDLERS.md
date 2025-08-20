# Documentação API - Sistema de URLs e Handlers

## Visão Geral
Sistema de URLs personalizadas, redirecionamentos, checkout embeddable e preview com Open Graph tags para compartilhamento otimizado em redes sociais.

## Endpoints Disponíveis

### 1. URL de Compartilhamento (Slug)
**GET** `/p/:slug`

**Autenticação:** Não requerida

**Descrição:** URL amigável para compartilhamento que redireciona para o checkout do produto.

**Parâmetros de Rota:**
- `slug` (string): Slug único do produto

**Exemplo de Uso:**
```
GET /p/curso-marketing-digital
```

**Comportamento:**
1. Busca o produto pelo slug
2. Registra analytics (`share_click`)
3. Redireciona para `/checkout/{slug}`

**Resposta de Sucesso (302):**
```
Location: /checkout/curso-marketing-digital
```

**Resposta de Erro (404):**
```html
<!DOCTYPE html>
<html>
<head>
  <title>Produto não encontrado</title>
  <meta charset="UTF-8">
</head>
<body>
  <h1>Produto não encontrado</h1>
  <p>O produto que você está procurando não existe ou foi removido.</p>
</body>
</html>
```

### 2. URL Curta (ID)
**GET** `/s/:id`

**Autenticação:** Não requerida

**Descrição:** URL curta baseada no ID do produto para compartilhamento compacto.

**Parâmetros de Rota:**
- `id` (string): ID único do produto

**Exemplo de Uso:**
```
GET /s/uuid-produto-123
```

**Comportamento:**
1. Busca o produto pelo ID
2. Registra analytics (`short_url_click`)
3. Redireciona para `/checkout/{product.slug}`

**Resposta de Sucesso (302):**
```
Location: /checkout/curso-marketing-digital
```

### 3. Checkout Embeddable
**GET** `/embed/:slug`

**Autenticação:** Não requerida

**Descrição:** Renderiza checkout minimalista para embedding em iframes de outros sites.

**Parâmetros de Rota:**
- `slug` (string): Slug único do produto

**Exemplo de Uso:**
```html
<iframe src="https://checkout.pro/embed/curso-marketing-digital" 
        width="450" height="600" frameborder="0">
</iframe>
```

**Resposta de Sucesso (200):**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Curso de Marketing Digital</title>
  <style>
    /* CSS otimizado para embed */
    .embed-container {
      max-width: 400px;
      margin: 0 auto;
      background: white;
      border-radius: 12px;
      box-shadow: 0 4px 20px rgba(0,0,0,0.1);
    }
    /* ... estilos completos ... */
  </style>
</head>
<body>
  <div class="embed-container">
    <div class="product-image">
      <img src="https://cdn.example.com/product.jpg" alt="Produto">
    </div>
    <div class="product-info">
      <div class="product-name">Curso de Marketing Digital</div>
      <div class="product-price">R$ 497,00</div>
      <div class="product-description">Aprenda marketing do zero ao avançado</div>
      <button class="buy-button" onclick="redirectToCheckout()">
        🛒 Comprar Agora
      </button>
      <div class="powered-by">Powered by CheckoutPro</div>
    </div>
  </div>
  
  <script>
    function redirectToCheckout() {
      window.open('/checkout/curso-marketing-digital', '_blank');
      // ... registrar analytics ...
    }
  </script>
</body>
</html>
```

**Características do Embed:**
- Design responsivo e minimalista
- Botão de compra abre checkout em nova aba
- Cores personalizáveis pelo produto
- Analytics automático
- Otimizado para performance

### 4. Preview com Open Graph
**GET** `/preview/:slug`

**Autenticação:** Não requerida

**Descrição:** Página de preview com meta tags otimizadas para compartilhamento em redes sociais.

**Parâmetros de Rota:**
- `slug` (string): Slug único do produto

**Exemplo de Uso:**
```
GET /preview/curso-marketing-digital
```

**Resposta de Sucesso (200):**
```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <title>Curso de Marketing Digital</title>
  
  <!-- Open Graph / Facebook -->
  <meta property="og:type" content="product">
  <meta property="og:url" content="https://checkout.pro/checkout/curso-marketing-digital">
  <meta property="og:title" content="Curso de Marketing Digital">
  <meta property="og:description" content="Aprenda marketing digital do zero ao avançado">
  <meta property="og:image" content="https://cdn.example.com/product.jpg">
  <meta property="product:price:amount" content="497.00">
  <meta property="product:price:currency" content="BRL">
  
  <!-- Twitter -->
  <meta property="twitter:card" content="summary_large_image">
  <meta property="twitter:url" content="https://checkout.pro/checkout/curso-marketing-digital">
  <meta property="twitter:title" content="Curso de Marketing Digital">
  <meta property="twitter:description" content="Aprenda marketing digital do zero ao avançado">
  <meta property="twitter:image" content="https://cdn.example.com/product.jpg">
  
  <!-- WhatsApp -->
  <meta property="og:site_name" content="CheckoutPro">
  <meta property="og:locale" content="pt_BR">
  
  <meta http-equiv="refresh" content="0; url=/checkout/curso-marketing-digital">
</head>
<body>
  <h1>Curso de Marketing Digital</h1>
  <p>Redirecionando para o checkout...</p>
  <a href="/checkout/curso-marketing-digital">Clique aqui se não for redirecionado automaticamente</a>
</body>
</html>
```

## Analytics Automático

### Eventos Registrados
Todos os acessos às URLs são automaticamente rastreados:

```javascript
await Analytics.create({
  product_id: product.id,
  event_type: 'share_click|short_url_click|embed_view|embed_checkout_click',
  session_id: req.sessionID || uuidv4(),
  user_agent: req.get('User-Agent'),
  ip_address: req.ip,
  referrer: req.get('Referer')
});
```

**Tipos de Eventos:**
- `share_click`: Clique em URL de compartilhamento (`/p/`)
- `short_url_click`: Clique em URL curta (`/s/`)
- `embed_view`: Visualização do embed
- `embed_checkout_click`: Clique no botão de compra do embed

## Integração e Uso

### URLs de Compartilhamento
```javascript
// Gerar URLs para compartilhamento
const generateShareUrls = (product) => {
  const baseUrl = 'https://checkout.pro';
  
  return {
    friendly: `${baseUrl}/p/${product.slug}`,
    short: `${baseUrl}/s/${product.id}`,
    embed: `${baseUrl}/embed/${product.slug}`,
    preview: `${baseUrl}/preview/${product.slug}`
  };
};

// Uso em componente de compartilhamento
const shareUrls = generateShareUrls(product);
console.log('URL amigável:', shareUrls.friendly);
console.log('URL curta:', shareUrls.short);
```

### Embedding em Sites
```html
<!-- Embed responsivo -->
<div style="width:100%;max-width:450px;margin:0 auto;">
  <iframe src="https://checkout.pro/embed/curso-marketing-digital"
          width="100%" height="600" frameborder="0"
          style="border-radius:12px;box-shadow:0 4px 20px rgba(0,0,0,0.1);">
  </iframe>
</div>

<!-- Embed com loading -->
<iframe src="https://checkout.pro/embed/curso-marketing-digital"
        width="450" height="600" frameborder="0"
        loading="lazy">
</iframe>
```

### Widget de Compartilhamento
```javascript
const createShareWidget = (product) => {
  const urls = generateShareUrls(product);
  
  return `
    <div class="share-widget">
      <h3>Compartilhar Produto</h3>
      
      <!-- WhatsApp -->
      <a href="https://wa.me/?text=${encodeURIComponent(urls.preview)}" 
         target="_blank" class="share-btn whatsapp">
        📱 WhatsApp
      </a>
      
      <!-- Facebook -->
      <a href="https://www.facebook.com/sharer/sharer.php?u=${encodeURIComponent(urls.preview)}"
         target="_blank" class="share-btn facebook">
        📘 Facebook
      </a>
      
      <!-- Twitter -->
      <a href="https://twitter.com/intent/tweet?url=${encodeURIComponent(urls.preview)}&text=${encodeURIComponent(product.name)}"
         target="_blank" class="share-btn twitter">
        🐦 Twitter
      </a>
      
      <!-- LinkedIn -->
      <a href="https://www.linkedin.com/sharing/share-offsite/?url=${encodeURIComponent(urls.preview)}"
         target="_blank" class="share-btn linkedin">
        💼 LinkedIn
      </a>
      
      <!-- Copiar Link -->
      <button onclick="copyToClipboard('${urls.friendly}')" class="share-btn copy">
        📋 Copiar Link
      </button>
    </div>
  `;
};
```

### Personalização do Embed
```javascript
// Personalizar cores do embed via configurações do produto
const product = {
  id: 'uuid-produto',
  name: 'Meu Curso',
  slug: 'meu-curso',
  price: 497.00,
  checkout_settings: {
    template: 'embed',
    primary_color: '#FF6B6B',  // Cor customizada
    secondary_color: '#4ECDC4'
  }
};

// O embed automaticamente usa as cores configuradas
```

## Open Graph Otimizado

### Tags Implementadas
```html
<!-- Básicas -->
<meta property="og:type" content="product">
<meta property="og:url" content="{checkout_url}">
<meta property="og:title" content="{product_name}">
<meta property="og:description" content="{product_description}">
<meta property="og:image" content="{product_image}">

<!-- Produto -->
<meta property="product:price:amount" content="{price}">
<meta property="product:price:currency" content="BRL">

<!-- Twitter Cards -->
<meta property="twitter:card" content="summary_large_image">
<meta property="twitter:title" content="{product_name}">
<meta property="twitter:description" content="{product_description}">
<meta property="twitter:image" content="{product_image}">

<!-- WhatsApp/Locale -->
<meta property="og:site_name" content="CheckoutPro">
<meta property="og:locale" content="pt_BR">
```

### Resultado do Compartilhamento
- **WhatsApp**: Mostra card com imagem, título, descrição e preço
- **Facebook**: Preview rico com botão de ação
- **Twitter**: Summary card com imagem grande
- **LinkedIn**: Preview profissional com detalhes

## Performance e SEO

### Otimizações Implementadas
1. **HTML Minimalista**: Código limpo e otimizado
2. **CSS Inline**: Para embeds sem dependências externas
3. **Lazy Loading**: Suporte a carregamento lazy para iframes
4. **Meta Tags Completas**: SEO otimizado para todas as plataformas
5. **Redirecionamento Rápido**: Redirect 302 para URLs de compartilhamento

### Cache e Performance
```javascript
// Headers de cache (implementar no servidor)
res.set({
  'Cache-Control': 'public, max-age=3600',  // 1 hora
  'Vary': 'User-Agent',
  'X-Content-Type-Options': 'nosniff'
});
```

## Monitoramento e Analytics

### Eventos Trackados
```javascript
// Dashboard de analytics pode consultar:
const shareStats = await Analytics.findAll({
  where: {
    product_id: productId,
    event_type: ['share_click', 'short_url_click', 'embed_view']
  },
  attributes: [
    'event_type',
    [sequelize.fn('COUNT', sequelize.col('id')), 'count'],
    [sequelize.fn('DATE', sequelize.col('created_at')), 'date']
  ],
  group: ['event_type', 'date']
});
```

### Métricas Importantes
- **CTR do Embed**: Taxa de clique no botão do embed
- **Origem do Tráfego**: Referrer das URLs acessadas
- **Dispositivos**: User-Agent analysis
- **Conversão**: Correlação entre cliques e vendas

## Casos de Uso

### 1. Site de Afiliados
```html
<!-- Embed em blog/site de afiliado -->
<article>
  <h2>Recomendo este curso</h2>
  <p>Este é o melhor curso que já vi sobre o assunto...</p>
  
  <iframe src="https://checkout.pro/embed/curso-recomendado"
          width="100%" height="600" frameborder="0">
  </iframe>
</article>
```

### 2. Email Marketing
```html
<!-- Link em email -->
<a href="https://checkout.pro/p/curso-especial?utm_source=email&utm_campaign=promo">
  🎯 Acesse a Oferta Especial
</a>
```

### 3. Redes Sociais
```javascript
// Post automatizado com preview otimizado
const shareText = `🚀 Novo curso disponível!\n\n${product.name}\n\nConfira: ${shareUrls.preview}`;

// O preview URL tem todas as meta tags para aparecer bonito
```

### 4. QR Code
```javascript
// Gerar QR Code com URL curta
const qrCodeUrl = `https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=${encodeURIComponent(shareUrls.short)}`;
```

## Observações Técnicas

1. **URLs Únicas**: Cada produto tem URLs exclusivas baseadas em slug e ID
2. **Fallback**: Páginas de erro amigáveis para produtos não encontrados
3. **Analytics Automático**: Todos os acessos são registrados automaticamente
4. **Responsivo**: Embeds funcionam em desktop e mobile
5. **Cross-Origin**: Headers apropriados para embedding seguro
6. **SEO Friendly**: Meta tags completas para todos os casos de uso
