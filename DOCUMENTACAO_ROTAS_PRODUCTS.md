# 📦 Documentação Completa - Rotas de Produtos

## Visão Geral
O arquivo `routes/products.js` contém todas as rotas para gerenciamento CRUD de produtos, incluindo configurações avançadas de checkout, temas, rastreamento, funil de vendas e muito mais. Todas as operações são protegidas e restritas ao usuário proprietário do produto.

## Middleware Utilizado
- **Autenticação**: `authenticateToken` para todas as rotas que modificam dados
- **Segurança**: Filtro rigoroso por `seller_id` do usuário logado
- **Validação**: Validação de slug, URLs personalizadas e dados obrigatórios

## URLs Reservadas do Sistema
O sistema possui uma lista de URLs reservadas que não podem ser usadas como slugs personalizados:
```javascript
['admin', 'api', 'auth', 'login', 'register', 'dashboard', 'profile', 'settings',
'checkout', 'payment', 'orders', 'products', 'analytics', 'reports', 'help',
'support', 'about', 'contact', 'terms', 'privacy', 'blog', 'docs', 'home',
'index', 'main', 'www', 'mail', 'ftp', 'test', 'dev', 'staging', 'prod',
'production', 'beta', 'alpha', 'demo', 'example', 'sample', 'template',
'static', 'assets', 'images', 'css', 'js', 'fonts', 'media', 'uploads',
'downloads', 'files', 'backup', 'tmp', 'temp', 'cache', 'log', 'logs']
```

---

## 📦 CRUD DE PRODUTOS

### GET /api/products
**Descrição**: Lista todos os produtos do usuário logado

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "products": [
    {
      "id": "uuid-produto-1",
      "name": "Curso de JavaScript Avançado",
      "description": "Aprenda JavaScript do básico ao avançado...",
      "price": 197.00,
      "slug": "abc123def-curso-javascript-avancado",
      "image_url": "https://exemplo.com/imagem.jpg",
      "image_upload_method": "url",
      "media_type": "digital",
      "category": "curso",
      "seller_id": "uuid-usuario",
      "status": "active",
      "tags": ["javascript", "programação", "web"],
      "theme_settings": {
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
      "tracking_settings": {
        "facebookPixel": "",
        "googleAnalytics": "",
        "googleTagManager": "",
        "tiktokPixel": "",
        "customPixels": [],
        "trackPurchases": true,
        "trackViews": true,
        "trackAddToCart": true
      },
      "payment_settings": {
        "acceptedMethods": ["credit_card", "pix"],
        "maxInstallments": 12,
        "pixDiscount": 5,
        "minInstallmentValue": 25,
        "acquirer": "default",
        "customAcquirer": null,
        "requirePhone": true,
        "requireAddress": true,
        "requireCPF": false
      },
      "seller": {
        "id": "uuid-usuario",
        "name": "João Silva",
        "email": "joao@email.com"
      },
      "urls": {
        "checkout": "http://localhost:3000/#/checkout/abc123def-curso-javascript-avancado",
        "preview": "http://localhost:3000/#/preview/abc123def-curso-javascript-avancado",
        "edit": "http://localhost:3000/#/products/edit/uuid-produto-1",
        "checkoutAPI": "http://localhost:5000/api/checkout/abc123def-curso-javascript-avancado"
      },
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2025-01-01T00:00:00Z"
    }
  ]
}
```

**Observações**:
- Lista apenas produtos do usuário logado (filtro por `seller_id`)
- Inclui informações do vendedor e URLs úteis
- Ordenação por data de criação (mais recentes primeiro)

---

### GET /api/products/:product_id
**Descrição**: Busca um produto específico por ID

**Autenticação**: Obrigatória

**Parâmetros**:
- `product_id` (string): UUID do produto

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "product": {
    "id": "uuid-produto-1",
    "name": "Curso de JavaScript Avançado",
    "description": "Aprenda JavaScript do básico ao avançado...",
    "price": 197.00,
    "slug": "abc123def-curso-javascript-avancado",
    "image_url": "https://exemplo.com/imagem.jpg",
    "media_type": "digital",
    "category": "curso",
    "status": "active",
    "seller": {
      "id": "uuid-usuario",
      "name": "João Silva",
      "email": "joao@email.com"
    },
    "urls": {
      "checkout": "http://localhost:3000/#/checkout/abc123def-curso-javascript-avancado",
      "preview": "http://localhost:3000/#/preview/abc123def-curso-javascript-avancado",
      "edit": "http://localhost:3000/#/products/edit/uuid-produto-1",
      "checkoutAPI": "http://localhost:5000/api/checkout/abc123def-curso-javascript-avancado"
    }
  }
}
```

**Erros Possíveis**:
- `404`: Produto não encontrado ou sem permissão
- `500`: Erro interno

---

### GET /api/products/slug/:slug
**Descrição**: Busca produto por slug ou URL personalizada (pública)

**Autenticação**: Não obrigatória

**Parâmetros**:
- `slug` (string): Slug do produto ou URL personalizada

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "product": {
    "id": "uuid-produto-1",
    "name": "Curso de JavaScript Avançado",
    "description": "Aprenda JavaScript do básico ao avançado...",
    "price": 197.00,
    "slug": "abc123def-curso-javascript-avancado",
    "sellerName": "João Silva",
    // ... outros campos do produto
  }
}
```

**Funcionalidades Especiais**:
- Busca primeiro por `slug`, depois por `customUrl` em `seo_settings`
- Detecta e corrige automaticamente slugs inválidos
- Gera novo slug se necessário

**Erros Possíveis**:
- `400`: Slug inválido (com código `INVALID_SLUG`)
- `404`: Produto não encontrado (com código `PRODUCT_NOT_FOUND`)
- `500`: Erro interno (com código `INTERNAL_ERROR`)

---

### GET /api/products/check-slug/:slug
**Descrição**: Verifica disponibilidade de slug/URL personalizada

**Autenticação**: Não obrigatória

**Parâmetros**:
- `slug` (string): Slug a ser verificado

**Query Parameters**:
- `exclude_id` (opcional): ID do produto a ser excluído da verificação

**Resposta de Sucesso (200)**:
```json
{
  "available": true,
  "errors": []
}
```

**Resposta com Erro de Validação (200)**:
```json
{
  "available": false,
  "errors": [
    "A URL deve ter pelo menos 3 caracteres",
    "A URL deve conter apenas letras, números, hífens e underscores"
  ]
}
```

**Regras de Validação**:
- Mínimo 3 caracteres, máximo 100
- Apenas letras, números, hífens e underscores
- Não pode começar ou terminar com hífen
- Não pode ter hífens consecutivos
- Não pode ser URL reservada do sistema

---

### POST /api/products
**Descrição**: Cria produto completo com todas as configurações

**Autenticação**: Obrigatória

**Headers**:
```
Authorization: Bearer jwt_token
Content-Type: application/json
```

**Dados Esperados**:
```json
{
  "name": "Curso de JavaScript Avançado",
  "description": "Aprenda JavaScript do básico ao avançado com projetos práticos",
  "price": 197.00,
  "imageUrl": "https://exemplo.com/imagem.jpg",
  "imageUploadMethod": "url",
  "mediaType": "digital",
  "category": "curso",
  "status": "active",
  "tags": ["javascript", "programação", "web"],
  
  "seoSettings": {
    "customUrl": "curso-javascript-avancado"
  },
  
  "themeSettings": {
    "template": "modern",
    "primaryColor": "#3B82F6",
    "secondaryColor": "#1E40AF",
    "backgroundColor": "#FFFFFF",
    "textColor": "#1F2937",
    "buttonStyle": "rounded",
    "fontFamily": "Inter"
  },
  
  "trackingSettings": {
    "facebookPixel": "123456789",
    "googleAnalytics": "GA-XXXXXXX",
    "trackPurchases": true,
    "trackViews": true
  },
  
  "paymentSettings": {
    "acceptedMethods": ["credit_card", "pix"],
    "maxInstallments": 12,
    "pixDiscount": 5,
    "requirePhone": true,
    "requireAddress": false,
    "requireCPF": false
  },
  
  "digitalDelivery": {
    "deliveryMethod": "email",
    "downloadLimit": 5,
    "downloadExpiry": 30,
    "files": [],
    "accessInstructions": "Instruções de acesso serão enviadas por email"
  },
  
  "courseData": {
    "title": "JavaScript do Zero ao Avançado",
    "description": "Curso completo de JavaScript",
    "instructorId": "será-forcado-para-usuario-logado",
    "duration": 40,
    "level": "intermediario",
    "language": "pt-BR"
  }
}
```

**Campos Obrigatórios**:
- `name`: Nome do produto
- `price`: Preço (número)

**Campos Especiais para Cursos**:
- Se `category` = "curso", `courseData` é obrigatório
- `courseData.title` é obrigatório para cursos
- `courseData.instructorId` é automaticamente definido como o usuário logado

**Geração de Slug**:
- Se `seoSettings.customUrl` fornecida: usa após validação
- Senão: gera automaticamente no formato `{id-único}-{nome-produto}`

**Resposta de Sucesso (201)**:
```json
{
  "success": true,
  "product": {
    "id": "uuid-novo-produto",
    "name": "Curso de JavaScript Avançado",
    "slug": "abc123def-curso-javascript-avancado",
    "price": 197.00,
    // ... todos os campos do produto criado
  },
  "course": {
    "id": "uuid-novo-curso",
    "product_id": "uuid-novo-produto",
    "title": "JavaScript do Zero ao Avançado",
    // ... dados do curso (se aplicável)
  }
}
```

**Validações**:
- URL personalizada única e válida
- Dados obrigatórios do curso (se categoria = "curso")
- Formato de preço válido

---

### PUT /api/products/:product_id
**Descrição**: Atualiza produto completo com todas as configurações

**Autenticação**: Obrigatória

**Parâmetros**:
- `product_id` (string): UUID do produto

**Headers**:
```
Authorization: Bearer jwt_token
Content-Type: application/json
```

**Dados Esperados**: Mesma estrutura do POST, campos opcionais

**Validações Especiais**:
- Verifica se nova URL personalizada está disponível
- Atualiza curso relacionado se aplicável
- Mantém configurações existentes para campos não fornecidos

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "product": {
    "id": "uuid-produto",
    "name": "Nome Atualizado",
    // ... dados atualizados
  }
}
```

---

### DELETE /api/products/:product_id
**Descrição**: Remove produto permanentemente

**Autenticação**: Obrigatória

**Parâmetros**:
- `product_id` (string): UUID do produto

**Headers**:
```
Authorization: Bearer jwt_token
```

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "Produto excluído com sucesso"
}
```

**Observações**:
- Remove permanentemente o produto
- Remove cursos relacionados automaticamente
- Operação irreversível

---

## ⚙️ CONFIGURAÇÕES ESPECÍFICAS

### PUT /api/products/:id/theme
**Descrição**: Atualiza apenas configurações de tema

**Autenticação**: Obrigatória

**Dados Esperados**:
```json
{
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
}
```

---

### PUT /api/products/:id/tracking
**Descrição**: Atualiza apenas configurações de rastreamento

**Autenticação**: Obrigatória

**Dados Esperados**:
```json
{
  "facebookPixel": "123456789",
  "googleAnalytics": "GA-XXXXXXX",
  "googleTagManager": "GTM-XXXXXXX",
  "tiktokPixel": "",
  "customPixels": [],
  "trackPurchases": true,
  "trackViews": true,
  "trackAddToCart": true
}
```

---

### PUT /api/products/:id/sales-funnel
**Descrição**: Atualiza configurações do funil de vendas

**Autenticação**: Obrigatória

**Dados Esperados**:
```json
{
  "orderBumps": [],
  "upsells": [],
  "downsells": [],
  "thankYouPage": {
    "enabled": true,
    "customMessage": "Obrigado pela compra!",
    "redirectUrl": "",
    "showSocialShare": true
  }
}
```

---

### PUT /api/products/:id/payment-settings
**Descrição**: Atualiza configurações de pagamento

**Autenticação**: Obrigatória

**Dados Esperados**:
```json
{
  "acceptedMethods": ["credit_card", "pix"],
  "maxInstallments": 12,
  "pixDiscount": 5,
  "minInstallmentValue": 25,
  "acquirer": "default",
  "customAcquirer": null,
  "requirePhone": true,
  "requireAddress": true,
  "requireCPF": false
}
```

---

## 📊 ESTATÍSTICAS E UTILITÁRIOS

### PATCH /api/products/:id/stats
**Descrição**: Atualiza estatísticas do produto (pública)

**Dados Esperados**:
```json
{
  "views": 1,
  "purchases": 1,
  "revenue": 197.00
}
```

---

### PUT /api/products/:id/template
**Descrição**: Atualiza template do produto (pública)

**Dados Esperados**:
```json
{
  "selectedTemplate": "modern",
  "themeSettings": {
    "primaryColor": "#3B82F6"
  }
}
```

---

### POST /api/products/fix-slugs
**Descrição**: Corrige todos os produtos com slugs inválidos (utilitário)

**Resposta de Sucesso (200)**:
```json
{
  "success": true,
  "message": "2 produtos corrigidos com sucesso",
  "fixes": [
    {
      "productId": "uuid-1",
      "productName": "Produto 1",
      "oldSlug": "undefined",
      "newSlug": "abc123def-produto-1"
    }
  ]
}
```

---

## 🔧 Funções Auxiliares

### Geração de Slug Único
- Formato: `{id-único-8-chars}-{nome-produto-normalizado}`
- Normalização: minúsculas, hífens para espaços, remove caracteres especiais
- Fallback para "produto" se nome inválido
- Verifica URLs reservadas
- Máximo 100 tentativas para garantir unicidade

### Validação de URL Personalizada
- Mínimo 3, máximo 100 caracteres
- Apenas letras, números, hífens e underscores
- Não inicia/termina com hífen
- Sem hífens consecutivos
- Não pode ser URL reservada

---

## 🛡️ Segurança

1. **Autenticação**: JWT obrigatório para operações de escrita
2. **Autorização**: Filtro rigoroso por `seller_id = req.user.id`
3. **Validação**: Dados obrigatórios e formatos válidos
4. **Sanitização**: Limpeza de URLs e campos de texto
5. **URLs Reservadas**: Proteção contra uso de URLs do sistema

---

## 📝 Estrutura Completa do Produto

Cada produto contém as seguintes configurações:

### Campos Básicos
- `id`, `name`, `description`, `price`, `slug`, `image_url`
- `media_type`, `category`, `seller_id`, `status`, `tags`

### Configurações Avançadas
- `theme_settings`: Aparência e estilo
- `tracking_settings`: Pixels e analytics
- `sales_funnel`: Upsells e order bumps
- `payment_settings`: Métodos e parcelamento
- `shipping_settings`: Entrega física
- `digital_delivery`: Entrega digital
- `meta_data`: SEO e metadados
- `automation_settings`: Automações e webhooks

### Configurações Legadas (Compatibilidade)
- `checkout_settings`, `seo_settings`, `digital_content`
- `physical_details`, `utm_data`, `email_settings`
- `discount_settings`, `analytics_settings`
- `members_area`, `selected_template`

---

**Próxima documentação**: `routes/orders.js` - Gestão de pedidos e checkout
