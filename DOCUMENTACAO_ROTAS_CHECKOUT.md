# 🛒 Documentação Completa - Rotas de Checkout

## Visão Geral
O arquivo `routes/checkout.js` contém o sistema completo de checkout unificado, incluindo páginas HTML personalizadas, APIs para SPAs, criação de pedidos, analytics e sistema completo de templates de checkout. O sistema é dividido em rotas públicas (para checkout) e rotas autenticadas (para gestão de templates).

## Arquitetura do Sistema
- **Rotas Públicas**: Acesso ao checkout, criação de pedidos e analytics
- **Rotas Autenticadas**: Gestão de templates, customização e administração
- **Sistema de Templates**: Layouts customizáveis com CSS e HTML gerados dinamicamente
- **Analytics Integrado**: Rastreamento de eventos e conversões

---

## 🛒 ROTAS PÚBLICAS DE CHECKOUT

### GET /checkout/:slug
**Descrição**: Página HTML completa de checkout com sistema de links únicos

**Autenticação**: Não obrigatória (pública)

**Parâmetros**:
- `slug` (string): Slug único do produto

**Query Parameters**:
- `template` (opcional): Template específico a usar

**Resposta de Sucesso (200)**: HTML completo da página de checkout

**Funcionalidades**:
- Gera HTML personalizado baseado nas configurações do produto
- Inclui pixels de rastreamento (Facebook, Google Analytics)
- Sistema responsivo com diferentes templates
- Formulário de checkout integrado
- Validações JavaScript em tempo real
- Registra visualização no analytics automaticamente

**Exemplo de URL**:
```
GET /checkout/abc123def-curso-javascript-avancado
GET /checkout/abc123def-curso-javascript-avancado?template=modern
```

**HTML Gerado**:
- Header com título, descrição e meta tags SEO
- Seção do produto com imagem, nome e preço
- Formulário de checkout com campos configuráveis
- Métodos de pagamento baseados nas configurações
- JavaScript para submissão do pedido
- Pixels de rastreamento integrados
- CSS customizado baseado no tema

**Campos do Formulário** (baseados em `payment_settings`):
- Nome completo (obrigatório)
- Email (obrigatório)
- Telefone (se `requirePhone: true`)
- CPF (se `requireCPF: true`)
- Endereço (se `requireAddress: true`)
- Método de pagamento (cartão, PIX, boleto)

**Erros Possíveis**:
- `404`: Produto não encontrado (página HTML de erro)
- `500`: Erro interno (página HTML de erro)

---

### GET /api/checkout/:slug
**Descrição**: API com dados completos do checkout para SPAs

**Autenticação**: Não obrigatória (pública)

**Parâmetros**:
- `slug` (string): Slug único do produto

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": {
    "product": {
      "id": "uuid-produto",
      "name": "Curso de JavaScript Avançado",
      "description": "Aprenda JavaScript do básico ao avançado...",
      "price": 197.00,
      "slug": "abc123def-curso-javascript-avancado",
      "imageUrl": "https://exemplo.com/imagem.jpg",
      "category": "curso",
      "status": "active",
      "tags": ["javascript", "programação"]
    },
    "seller": {
      "id": "uuid-vendedor",
      "name": "João Silva",
      "email": "joao@email.com",
      "avatar": "https://exemplo.com/avatar.jpg"
    },
    "theme": {
      "template": "modern",
      "primaryColor": "#3B82F6",
      "secondaryColor": "#1E40AF",
      "backgroundColor": "#FFFFFF",
      "textColor": "#1F2937",
      "buttonStyle": "rounded",
      "fontFamily": "Inter",
      "logoUrl": "",
      "backgroundImage": "",
      "customCSS": ""
    },
    "tracking": {
      "facebookPixel": "123456789",
      "googleAnalytics": "GA-XXXXXXX",
      "googleTagManager": "",
      "tiktokPixel": "",
      "customPixels": [],
      "trackPurchases": true,
      "trackViews": true,
      "trackAddToCart": true
    },
    "payment": {
      "acceptedMethods": ["credit_card", "pix"],
      "maxInstallments": 12,
      "pixDiscount": 5,
      "minInstallmentValue": 25,
      "requirePhone": true,
      "requireAddress": true,
      "requireCPF": false
    },
    "salesFunnel": {
      "orderBumps": [],
      "upsells": [],
      "downsells": [],
      "thankYouPage": {
        "enabled": true,
        "customMessage": "Obrigado pela compra!",
        "redirectUrl": "",
        "showSocialShare": true
      }
    },
    "shipping": {
      "enabled": false,
      "freeShipping": false,
      "estimatedDelivery": "5-10 dias úteis"
    },
    "digitalDelivery": {
      "deliveryMethod": "email",
      "downloadLimit": 5,
      "downloadExpiry": 30,
      "accessInstructions": "Instruções por email"
    },
    "meta": {
      "metaTitle": "Curso de JavaScript Avançado",
      "metaDescription": "Aprenda JavaScript...",
      "ogImage": "https://exemplo.com/og-image.jpg"
    },
    "urls": {
      "checkout": "/checkout/abc123def-curso-javascript-avancado",
      "api": "/api/checkout/abc123def-curso-javascript-avancado",
      "order": "/api/checkout/abc123def-curso-javascript-avancado/order",
      "analytics": "/api/checkout/abc123def-curso-javascript-avancado/analytics"
    }
  }
}
```

**Uso**: Ideal para SPAs React, Vue, Angular que precisam dos dados para renderizar checkout customizado

**Erros Possíveis**:
- `404`: Produto não encontrado
- `500`: Erro interno

---

### POST /api/checkout/:slug/order
**Descrição**: Cria novo pedido no sistema

**Autenticação**: Não obrigatória (pública)

**Parâmetros**:
- `slug` (string): Slug único do produto

**Dados Esperados**:
```json
{
  "customer": {
    "name": "João Silva",
    "email": "joao@email.com",
    "phone": "(11) 99999-9999",
    "document": "123.456.789-00"
  },
  "paymentMethod": "credit_card",
  "installments": 3,
  "totalAmount": 197.00,
  "orderBumps": [
    {
      "id": "bump-1",
      "name": "E-book Bônus",
      "price": 29.90
    }
  ],
  "utm": {
    "source": "facebook",
    "medium": "cpc",
    "campaign": "lancamento"
  }
}
```

**Campos Obrigatórios**:
- `customer.name`: Nome do cliente
- `customer.email`: Email do cliente

**Campos Opcionais**:
- `customer.phone`: Telefone (obrigatório se `requirePhone: true`)
- `customer.document`: CPF/CNPJ (obrigatório se `requireCPF: true`)
- `paymentMethod`: Método de pagamento selecionado
- `installments`: Número de parcelas (padrão: 1)
- `totalAmount`: Valor total (padrão: preço do produto)
- `orderBumps`: Array de order bumps selecionados
- `utm`: Dados de UTM para rastreamento

**Resposta de Sucesso (201)**:
```json
{
  "success": true,
  "message": "Pedido criado com sucesso",
  "data": {
    "orderId": "uuid-pedido",
    "status": "pending",
    "totalAmount": 226.90,
    "paymentUrl": "/payment/uuid-pedido",
    "trackingUrl": "/orders/uuid-pedido/track"
  }
}
```

**Funcionalidades Automáticas**:
- Registra evento `order_created` no analytics
- Calcula valor total incluindo order bumps
- Gera URLs de pagamento e rastreamento
- Valida dados obrigatórios baseados nas configurações do produto

**Erros Possíveis**:
- `400`: Dados obrigatórios ausentes
- `404`: Produto não encontrado
- `500`: Erro ao processar pedido

---

### POST /api/checkout/:slug/analytics
**Descrição**: Registra eventos de analytics

**Autenticação**: Não obrigatória (pública)

**Parâmetros**:
- `slug` (string): Slug único do produto

**Dados Esperados**:
```json
{
  "event": "add_to_cart",
  "data": {
    "timestamp": 1704067200000,
    "value": 197.00,
    "currency": "BRL",
    "custom_data": {
      "source": "facebook_ad",
      "campaign_id": "123456"
    }
  }
}
```

**Eventos Comuns**:
- `checkout_view`: Visualização da página
- `add_to_cart`: Adição ao carrinho
- `begin_checkout`: Início do checkout
- `payment_info_added`: Informações de pagamento adicionadas
- `purchase`: Compra finalizada
- `form_submit`: Envio do formulário

**Resposta de Sucesso (200)**:
```json
{
  "success": true
}
```

**Uso**: Rastreamento detalhado de comportamento do usuário para otimização de conversão

---

## 🔧 ROTAS AUTENTICADAS DE TEMPLATES

### GET /api/checkout/templates
**Descrição**: Lista templates do usuário

**Autenticação**: Obrigatória (Producer/Admin)

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid-template",
      "name": "Template Moderno",
      "layout": "modern",
      "description": "Template moderno para produtos digitais",
      "theme": {
        "primaryColor": "#3B82F6",
        "backgroundColor": "#FFFFFF",
        "fontFamily": "Inter"
      },
      "isDefault": false,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2025-01-01T00:00:00Z"
    }
  ]
}
```

---

### GET /api/checkout/templates/default
**Descrição**: Busca template padrão do usuário

**Autenticação**: Obrigatória (Producer/Admin)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": {
    "id": "uuid-template-default",
    "name": "Meu Template Padrão",
    "layout": "classic",
    "isDefault": true,
    // ... outras propriedades
  }
}
```

---

### POST /api/checkout/templates
**Descrição**: Cria novo template

**Autenticação**: Obrigatória (Producer/Admin)

**Dados Esperados**:
```json
{
  "name": "Meu Template Premium",
  "layout": "premium",
  "description": "Template elegante para produtos premium",
  "theme": {
    "primaryColor": "#8B5CF6",
    "secondaryColor": "#7C3AED",
    "backgroundColor": "#FAFAFA",
    "textColor": "#1F2937",
    "buttonStyle": "pill",
    "fontFamily": "Poppins",
    "logoUrl": "https://exemplo.com/logo.png",
    "backgroundImage": "",
    "customCSS": ".custom-style { color: red; }"
  },
  "customizations": {
    "headerLayout": "centered",
    "showSecurityBadges": true,
    "showTestimonials": false,
    "enableCountdownTimer": true
  }
}
```

**Layouts Disponíveis**:
- `classic`: Layout tradicional e limpo
- `modern`: Design contemporâneo
- `minimal`: Design clean e minimalista
- `premium`: Layout elegante para produtos premium
- `custom`: Layout totalmente customizável

**Resposta de Sucesso (201)**:
```json
{
  "success": true,
  "message": "Template criado com sucesso",
  "data": {
    "id": "uuid-novo-template",
    "name": "Meu Template Premium",
    "layout": "premium",
    // ... dados completos do template
  }
}
```

**Validações**:
- Nome obrigatório
- Layout deve ser válido
- Cores em formato hexadecimal
- CSS customizado validado

---

### PUT /api/checkout/templates/:id
**Descrição**: Atualiza template existente

**Autenticação**: Obrigatória (Producer/Admin)

**Parâmetros**:
- `id` (string): UUID do template

**Dados Esperados**: Mesma estrutura do POST, campos opcionais

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Template atualizado com sucesso",
  "data": {
    // ... dados atualizados do template
  }
}
```

**Segurança**: Verifica ownership do template pelo usuário

---

### POST /api/checkout/templates/:id/clone
**Descrição**: Clona template existente

**Autenticação**: Obrigatória (Producer/Admin)

**Parâmetros**:
- `id` (string): UUID do template a ser clonado

**Dados Esperados**:
```json
{
  "newName": "Cópia do Template Premium"
}
```

**Resposta de Sucesso (201)**:
```json
{
  "success": true,
  "message": "Template clonado com sucesso",
  "data": {
    "id": "uuid-template-clonado",
    "name": "Cópia do Template Premium",
    // ... dados do template clonado
  }
}
```

---

### GET /api/checkout/templates/:id/css
**Descrição**: Gera CSS do template

**Autenticação**: Obrigatória (Producer/Admin)

**Parâmetros**:
- `id` (string): UUID do template

**Resposta de Sucesso (200)**:
```css
/* CSS gerado dinamicamente baseado no template */
:root {
  --primary-color: #8B5CF6;
  --secondary-color: #7C3AED;
  --bg-color: #FAFAFA;
}

.checkout-container {
  background: var(--bg-color);
  font-family: 'Poppins', sans-serif;
}

.checkout-button {
  background: var(--primary-color);
  border-radius: 2rem;
}

/* Customizações específicas do usuário */
.custom-style { color: red; }
```

**Content-Type**: `text/css`

---

### GET /api/checkout/templates/:id/html
**Descrição**: Gera preview HTML do template

**Autenticação**: Obrigatória (Producer/Admin)

**Parâmetros**:
- `id` (string): UUID do template

**Query Parameters**:
- `productData` (opcional): JSON com dados do produto para preview

**Resposta de Sucesso (200)**: HTML completo da página

**Content-Type**: `text/html`

---

### POST /api/checkout/templates/:id/preview
**Descrição**: Gera dados de preview do template

**Autenticação**: Obrigatória (Producer/Admin)

**Parâmetros**:
- `id` (string): UUID do template

**Dados Esperados**:
```json
{
  "productData": {
    "name": "Produto de Exemplo",
    "price": "99.90",
    "description": "Descrição para preview",
    "image_url": "https://via.placeholder.com/300x200"
  },
  "deviceType": "desktop"
}
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": {
    "template": {
      // ... dados do template
    },
    "product": {
      // ... dados do produto para preview
    },
    "css": "/* CSS gerado */",
    "deviceType": "desktop"
  }
}
```

---

### DELETE /api/checkout/templates/:id
**Descrição**: Remove template

**Autenticação**: Obrigatória (Producer/Admin)

**Parâmetros**:
- `id` (string): UUID do template

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Template excluído com sucesso"
}
```

---

### GET /api/checkout/layouts
**Descrição**: Lista layouts disponíveis

**Autenticação**: Obrigatória (Producer/Admin)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": [
    {
      "id": "classic",
      "name": "Clássico",
      "description": "Layout tradicional e limpo",
      "preview": "/assets/layouts/classic-preview.png"
    },
    {
      "id": "modern",
      "name": "Moderno",
      "description": "Design contemporâneo com elementos visuais modernos",
      "preview": "/assets/layouts/modern-preview.png"
    },
    {
      "id": "minimal",
      "name": "Minimalista",
      "description": "Design clean e minimalista",
      "preview": "/assets/layouts/minimal-preview.png"
    },
    {
      "id": "premium",
      "name": "Premium",
      "description": "Layout elegante para produtos premium",
      "preview": "/assets/layouts/premium-preview.png"
    },
    {
      "id": "custom",
      "name": "Personalizado",
      "description": "Layout totalmente customizável",
      "preview": "/assets/layouts/custom-preview.png"
    }
  ]
}
```

---

### GET /api/checkout/templates/presets
**Descrição**: Lista templates predefinidos

**Autenticação**: Obrigatória (Producer/Admin)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "data": [
    {
      "id": "modern_blue",
      "name": "Moderno Azul",
      "layout": "modern",
      "theme": {
        "primaryColor": "#3B82F6",
        "backgroundColor": "#FFFFFF"
      },
      "description": "Template moderno com esquema azul"
    }
    // ... outros presets
  ]
}
```

---

## 🎨 Sistema de Templates

### Estrutura do Template
Cada template contém:

**Configurações Básicas**:
- `name`: Nome do template
- `layout`: Layout base (classic, modern, minimal, premium, custom)
- `description`: Descrição do template
- `isDefault`: Se é o template padrão do usuário

**Configurações de Tema**:
- `primaryColor`: Cor primária
- `secondaryColor`: Cor secundária
- `backgroundColor`: Cor de fundo
- `textColor`: Cor do texto
- `buttonStyle`: Estilo dos botões (rounded, pill, square)
- `fontFamily`: Família da fonte
- `logoUrl`: URL do logo
- `backgroundImage`: Imagem de fundo
- `customCSS`: CSS personalizado

**Customizações Avançadas**:
- `headerLayout`: Layout do cabeçalho
- `showSecurityBadges`: Mostrar selos de segurança
- `showTestimonials`: Mostrar depoimentos
- `enableCountdownTimer`: Habilitar timer de urgência

### Geração Dinâmica
- **CSS**: Gerado dinamicamente baseado nas configurações
- **HTML**: Template base + customizações + dados do produto
- **JavaScript**: Funcionalidades interativas incluídas automaticamente

---

## 🔒 Segurança e Permissões

### Rotas Públicas
- Acesso livre para checkout e criação de pedidos
- Rate limiting aplicado para prevenir spam
- Validação de dados de entrada
- Sanitização de outputs HTML

### Rotas Autenticadas
- JWT obrigatório
- Verificação de role (Producer/Admin)
- Ownership de templates verificado
- Logs de atividade para auditoria

### Validações
- Templates: Nome obrigatório, layout válido
- CSS: Sanitização de código personalizado
- Dados de pedido: Campos obrigatórios baseados em configurações
- Analytics: Estrutura de eventos validada

---

## 📊 Analytics e Rastreamento

### Eventos Automáticos
- `checkout_view`: Visualização da página
- `order_created`: Pedido criado

### Eventos Customizados
- Qualquer evento pode ser registrado via API
- Estrutura flexível de dados
- Rastreamento de sessão e IP

### Pixels Integrados
- **Facebook Pixel**: Eventos automáticos de ViewContent
- **Google Analytics**: Pageviews e conversões
- **Google Tag Manager**: Integração completa
- **TikTok Pixel**: Suporte adicional

---

## 🚀 Funcionalidades Avançadas

### Sistema de Links Únicos
- Cada produto tem slug único e imutável
- URLs amigáveis e SEO-friendly
- Redirecionamento automático para URLs corretas

### Personalização Visual
- Templates responsivos
- CSS gerado dinamicamente
- Suporte a temas escuros/claros
- Fontes personalizadas

### Otimização de Conversão
- Order bumps integrados
- Descontos por método de pagamento
- Selos de segurança
- Urgência e escassez

### Formulários Inteligentes
- Campos obrigatórios baseados em configurações
- Validação em tempo real
- Máscaras automáticas para CPF/telefone
- Preenchimento automático

---

**Próxima documentação**: `routes/orders.js` - Gestão de pedidos e pagamentos
