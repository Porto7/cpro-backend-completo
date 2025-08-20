# Documentação - Sistema Frontend (routes/frontend.js)

## Visão Geral
Sistema especializado para frontend que fornece APIs otimizadas para geração de URLs, configurações de checkout, estatísticas de dashboard e dados de produtos formatados especificamente para consumo frontend.

## Características Principais
- **Geração Automática de URLs**: Múltiplas URLs para produtos e checkout
- **Configurações Dinâmicas**: Settings personalizáveis para checkout
- **Estatísticas Rápidas**: Dashboard metrics otimizadas
- **Preview de Produtos**: Dados formatados para listagens frontend
- **UTM Tracking**: Suporte completo para parâmetros de rastreamento

## Rotas Implementadas

### 1. Geração de URLs de Produto

#### `GET /api/frontend/products/:id/urls`
**Descrição**: Gera todas as URLs relacionadas a um produto
**Acesso**: Privado (token required)

**Response Success**:
```json
{
  "success": true,
  "data": {
    "product": {
      "id": "prod-123",
      "name": "Curso de Marketing Digital",
      "slug": "curso-marketing-digital"
    },
    "urls": {
      "product_url": "https://api.exemplo.com/api/products/slug/curso-marketing-digital",
      "checkout_url": "https://api.exemplo.com/checkout/curso-marketing-digital",
      "checkout_premium_url": "https://api.exemplo.com/checkout/curso-marketing-digital?template=premium",
      "checkout_modern_url": "https://api.exemplo.com/checkout/curso-marketing-digital?template=modern",
      "checkout_classic_url": "https://api.exemplo.com/checkout/curso-marketing-digital?template=classic",
      "api_checkout": "https://api.exemplo.com/api/checkout/curso-marketing-digital",
      "api_product": "https://api.exemplo.com/api/products/slug/curso-marketing-digital",
      "embed_url": "https://api.exemplo.com/embed/curso-marketing-digital",
      "analytics_url": "https://api.exemplo.com/api/analytics/products/prod-123/stats",
      "share_url": "https://api.exemplo.com/p/curso-marketing-digital",
      "short_url": "https://api.exemplo.com/s/prod-123"
    }
  }
}
```

**Tipos de URLs Geradas**:
- **product_url**: URL da API do produto
- **checkout_url**: URL base do checkout
- **checkout_*_url**: URLs com templates específicos
- **api_checkout**: Endpoint de API do checkout
- **embed_url**: URL para embed em iframes
- **analytics_url**: URL das analytics do produto
- **share_url**: URL amigável para compartilhamento
- **short_url**: URL encurtada

#### `POST /api/frontend/products/:id/generate-checkout-url`
**Descrição**: Gera URL de checkout personalizada com parâmetros UTM e configurações
**Acesso**: Privado (token required)

```javascript
// Payload de exemplo
{
  "template": "premium",
  "utm_params": {
    "utm_source": "google",
    "utm_medium": "cpc",
    "utm_campaign": "campanha-verao-2024",
    "utm_content": "anuncio-principal",
    "utm_term": "marketing-digital"
  },
  "custom_settings": {
    "theme_color": "#FF6B35",
    "show_testimonials": true,
    "hide_quantity": false,
    "auto_apply_coupon": "DESCONTO10"
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "data": {
    "checkout_url": "https://api.exemplo.com/checkout/curso-marketing-digital?utm_source=google&utm_medium=cpc&utm_campaign=campanha-verao-2024&utm_content=anuncio-principal&utm_term=marketing-digital&template=premium&settings=eyJ0aGVtZV9jb2xvciI6IiNGRjZCMzUifQ==",
    "qr_code_data": "https://api.exemplo.com/checkout/curso-marketing-digital?utm_source=google...",
    "short_code": "prod-123",
    "expires_in": null
  }
}
```

**Parâmetros Suportados**:
- **template**: Template do checkout (modern, classic, premium)
- **utm_params**: Todos os parâmetros UTM padrão
- **custom_settings**: Configurações personalizadas (codificadas em base64)

### 2. Configurações de Checkout

#### `GET /api/frontend/checkout-config`
**Descrição**: Obtém configurações globais de checkout para o frontend
**Acesso**: Privado (token required)

**Response Success**:
```json
{
  "success": true,
  "data": {
    "payment_methods": {
      "credit_card": {
        "enabled": true,
        "max_installments": 12,
        "min_installment_value": 10.00
      },
      "pix": {
        "enabled": true,
        "discount_percentage": 5
      },
      "paypal": {
        "enabled": true
      },
      "mercadopago": {
        "enabled": false
      }
    },
    "checkout_templates": [
      {
        "id": "modern",
        "name": "Moderno",
        "description": "Design limpo e moderno"
      },
      {
        "id": "classic",
        "name": "Clássico",
        "description": "Design tradicional"
      },
      {
        "id": "premium",
        "name": "Premium",
        "description": "Design premium com animações"
      }
    ],
    "form_fields": {
      "available": ["name", "email", "phone", "cpf", "address"],
      "required": ["name", "email"],
      "validation": {
        "email": "^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$",
        "phone": "^\\+?[1-9]\\d{1,14}$",
        "cpf": "^\\d{11}$"
      }
    },
    "analytics": {
      "google_analytics": "GA_TRACKING_ID",
      "facebook_pixel": "FB_PIXEL_ID",
      "events_tracking": true
    }
  }
}
```

#### `POST /api/frontend/products/:id/checkout-settings`
**Descrição**: Atualiza configurações específicas de checkout do produto
**Acesso**: Privado (token required)

```javascript
// Payload de exemplo
{
  "template": "premium",
  "theme_color": "#FF6B35",
  "show_testimonials": true,
  "testimonials": [
    {
      "name": "João Silva",
      "text": "Produto incrível, recomendo!",
      "rating": 5
    }
  ],
  "custom_fields": [
    {
      "name": "empresa",
      "label": "Nome da Empresa",
      "type": "text",
      "required": false
    }
  ],
  "payment_settings": {
    "pix_discount": 10,
    "max_installments": 6
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Configurações de checkout atualizadas",
  "data": {
    "product_id": "prod-123",
    "checkout_settings": {
      "template": "premium",
      "theme_color": "#FF6B35",
      "show_testimonials": true,
      "testimonials": [...],
      "custom_fields": [...],
      "payment_settings": {...}
    }
  }
}
```

### 3. Estatísticas do Dashboard

#### `GET /api/frontend/dashboard/quick-stats`
**Descrição**: Estatísticas rápidas e otimizadas para dashboard frontend
**Acesso**: Privado (token required)

**Response Success**:
```json
{
  "success": true,
  "data": {
    "total_products": 15,
    "total_orders": 1247,
    "total_revenue": 89750.50,
    "pending_orders": 23,
    "conversion_rate": 3.45,
    "top_product": {
      "id": "prod-456",
      "name": "Curso de Marketing Digital Avançado",
      "slug": "curso-marketing-digital-avancado",
      "total_sales": 456,
      "total_revenue": 34200.00
    }
  }
}
```

**Métricas Incluídas**:
- **total_products**: Total de produtos do usuário
- **total_orders**: Total de pedidos realizados
- **total_revenue**: Receita total de pedidos completados
- **pending_orders**: Pedidos pendentes
- **conversion_rate**: Taxa de conversão (pedidos/visualizações)
- **top_product**: Produto com mais vendas

### 4. Preview de Produtos

#### `GET /api/frontend/products/preview`
**Descrição**: Lista produtos com dados otimizados para frontend
**Acesso**: Privado (token required)

**Query Parameters**:
- `page`: Página (padrão: 1)
- `limit`: Itens por página (padrão: 10)
- `status`: Filtro de status (all, active, inactive, draft)

**Response Success**:
```json
{
  "success": true,
  "data": {
    "products": [
      {
        "id": "prod-123",
        "name": "Curso de Marketing Digital",
        "slug": "curso-marketing-digital",
        "price": 197.00,
        "status": "active",
        "image_url": "https://exemplo.com/imagem.jpg",
        "created_at": "2024-01-01T10:00:00Z",
        "checkout_settings": {
          "template": "modern",
          "theme_color": "#007bff"
        },
        "checkout_url": "https://api.exemplo.com/checkout/curso-marketing-digital",
        "api_url": "https://api.exemplo.com/api/checkout/curso-marketing-digital",
        "share_url": "https://api.exemplo.com/p/curso-marketing-digital"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 15,
      "pages": 2
    }
  }
}
```

**Dados Otimizados Incluídos**:
- Informações básicas do produto
- URLs geradas automaticamente
- Configurações de checkout
- Paginação completa

## Funcionalidades Avançadas

### 1. UTM Tracking Automático
- Suporte completo para parâmetros UTM
- Preservação de tracking em todas as URLs
- Integração com analytics

### 2. Configurações Dinâmicas
- Settings codificadas em base64 para URLs
- Merge inteligente com configurações existentes
- Validação de configurações

### 3. URLs Inteligentes
- Geração automática baseada no host
- Múltiplos formatos para diferentes usos
- Suporte para templates e parâmetros

### 4. Performance Otimizada
- Consultas SQL otimizadas para dashboard
- Paginação eficiente
- Caching de configurações

### 5. Templates de Checkout
- **Modern**: Design limpo e responsivo
- **Classic**: Layout tradicional
- **Premium**: Design avançado com animações

## Integração Frontend

### React/Vue.js Example
```javascript
// Buscar URLs do produto
const getProductUrls = async (productId) => {
  const response = await fetch(`/api/frontend/products/${productId}/urls`, {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });
  const data = await response.json();
  return data.data.urls;
};

// Gerar URL personalizada
const generateCheckoutUrl = async (productId, config) => {
  const response = await fetch(`/api/frontend/products/${productId}/generate-checkout-url`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`
    },
    body: JSON.stringify(config)
  });
  const data = await response.json();
  return data.data.checkout_url;
};

// Buscar estatísticas do dashboard
const getDashboardStats = async () => {
  const response = await fetch('/api/frontend/dashboard/quick-stats', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });
  const data = await response.json();
  return data.data;
};
```

### URL Builder Helper
```javascript
class CheckoutUrlBuilder {
  constructor(baseUrl, productSlug) {
    this.baseUrl = baseUrl;
    this.productSlug = productSlug;
    this.params = new URLSearchParams();
  }
  
  setTemplate(template) {
    this.params.set('template', template);
    return this;
  }
  
  setUTM(source, medium, campaign, content = null, term = null) {
    this.params.set('utm_source', source);
    this.params.set('utm_medium', medium);
    this.params.set('utm_campaign', campaign);
    if (content) this.params.set('utm_content', content);
    if (term) this.params.set('utm_term', term);
    return this;
  }
  
  setSettings(settings) {
    this.params.set('settings', btoa(JSON.stringify(settings)));
    return this;
  }
  
  build() {
    const url = `${this.baseUrl}/checkout/${this.productSlug}`;
    const paramString = this.params.toString();
    return paramString ? `${url}?${paramString}` : url;
  }
}

// Uso
const url = new CheckoutUrlBuilder('https://api.exemplo.com', 'meu-produto')
  .setTemplate('premium')
  .setUTM('google', 'cpc', 'campanha-2024')
  .setSettings({ theme_color: '#FF6B35' })
  .build();
```

## Configurações de Checkout Avançadas

### Configurações de Template
```javascript
{
  "template": "premium",
  "theme_color": "#FF6B35",
  "background_color": "#FFFFFF",
  "text_color": "#333333",
  "button_color": "#28a745",
  "show_testimonials": true,
  "show_guarantee": true,
  "show_timer": false,
  "custom_css": ".checkout-form { border-radius: 10px; }"
}
```

### Configurações de Pagamento
```javascript
{
  "payment_methods": {
    "credit_card": true,
    "pix": true,
    "paypal": false
  },
  "pix_discount": 15,
  "max_installments": 8,
  "min_installment_value": 25.00,
  "auto_apply_coupons": ["DESCONTO10", "PRIMEIRA_COMPRA"]
}
```

### Campos Personalizados
```javascript
{
  "custom_fields": [
    {
      "name": "telefone_adicional",
      "label": "Telefone Adicional",
      "type": "tel",
      "required": false,
      "placeholder": "(11) 99999-9999"
    },
    {
      "name": "como_conheceu",
      "label": "Como nos conheceu?",
      "type": "select",
      "options": ["Google", "Facebook", "Indicação", "Outros"],
      "required": true
    }
  ]
}
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Parâmetros inválidos ou configurações mal formadas |
| 401 | Token de autenticação inválido |
| 404 | Produto não encontrado ou sem permissão |
| 500 | Erro interno do servidor |

## Considerações de Performance

1. **Caching**: URLs e configurações em cache
2. **Consultas Otimizadas**: JOINs eficientes para estatísticas
3. **Paginação**: Limite de 100 itens por página
4. **Compressão**: Settings em base64 para URLs compactas
5. **Lazy Loading**: Carregamento sob demanda de dados

## Segurança

1. **Autenticação**: JWT token obrigatório
2. **Ownership**: Verificação de propriedade dos produtos
3. **Validação**: Sanitização de parâmetros UTM
4. **Rate Limiting**: Proteção contra abuse
5. **CORS**: Configuração adequada para frontend

Este sistema frontend oferece uma API completa e otimizada para criação de interfaces de usuário modernas e eficientes, com foco em geração de URLs inteligentes e configurações flexíveis de checkout.
