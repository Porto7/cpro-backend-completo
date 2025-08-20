# Documentação - Sistema de Funil de Vendas (routes/funnel.js)

## Visão Geral
Sistema completo de funil de vendas que permite criar estratégias avançadas de conversão através de Order Bumps, Upsells e Back Redirects para maximizar o valor do carrinho e reduzir abandono.

## Funcionalidades Principais
- **Order Bumps**: Ofertas adicionais durante o checkout
- **Upsells**: Sequência de ofertas pós-compra
- **Back Redirects**: Recuperação de carrinho abandonado
- **Tracking de Conversões**: Monitoramento detalhado de performance
- **Templates Customizáveis**: Diferentes layouts para cada estratégia

## Rotas Implementadas

### 1. Order Bumps

#### `GET /api/funnel/order-bumps`
**Descrição**: Lista order bumps por produto
**Acesso**: Producer, Admin
**Query Parameters**: `productId` (obrigatório)

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "id": "bump-123",
      "name": "Bônus: Planilhas de Marketing",
      "main_product_id": "prod-456",
      "bump_product_id": "prod-789",
      "position": "before_payment",
      "title": "🎯 Adicione por apenas R$ 47,00",
      "description": "Ganhe acesso às planilhas exclusivas de marketing que uso no meu negócio",
      "image_url": "https://exemplo.com/planilhas.jpg",
      "discount_percentage": 50,
      "original_price": 97.00,
      "bump_price": 47.00,
      "is_active": true,
      "conversion_rate": 23.5,
      "total_conversions": 156,
      "created_at": "2024-01-01T10:00:00Z"
    }
  ]
}
```

#### `POST /api/funnel/order-bumps`
**Descrição**: Cria novo order bump
**Acesso**: Producer, Admin

```javascript
// Payload de exemplo
{
  "name": "Bônus: Planilhas de Marketing",
  "main_product_id": "prod-456",
  "bump_product_id": "prod-789",
  "position": "before_payment", // before_payment, after_payment, sidebar
  "title": "🎯 Adicione por apenas R$ 47,00",
  "description": "Ganhe acesso às planilhas exclusivas de marketing que uso no meu negócio",
  "image_url": "https://exemplo.com/planilhas.jpg",
  "discount_percentage": 50,
  "is_default_checked": false,
  "display_settings": {
    "background_color": "#f8f9fa",
    "border_color": "#28a745",
    "text_color": "#212529",
    "highlight_color": "#ffc107"
  },
  "triggers": {
    "show_for_all": true,
    "min_cart_value": 0,
    "utm_sources": ["google", "facebook"]
  }
}
```

**Posições Disponíveis**:
- `before_payment`: Antes do formulário de pagamento
- `after_payment`: Após os dados de pagamento
- `sidebar`: Na barra lateral do checkout

**Response Success**:
```json
{
  "success": true,
  "message": "Order bump criado com sucesso",
  "data": {
    "id": "bump-123",
    "name": "Bônus: Planilhas de Marketing",
    "main_product_id": "prod-456",
    "bump_product_id": "prod-789",
    "position": "before_payment",
    "is_active": true,
    "created_at": "2024-01-01T10:00:00Z"
  }
}
```

### 2. Upsells

#### `GET /api/funnel/upsells`
**Descrição**: Lista sequência de upsells por produto
**Acesso**: Producer, Admin
**Query Parameters**: `productId` (obrigatório)

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "id": "upsell-456",
      "name": "Upsell 1: Consultoria Individual",
      "trigger_product_id": "prod-123",
      "upsell_product_id": "prod-789",
      "sequence_order": 1,
      "template": "video",
      "headline": "🚀 Acelere Seus Resultados com Consultoria 1:1",
      "subheadline": "Apenas para os primeiros 20 alunos",
      "video_url": "https://exemplo.com/video-upsell.mp4",
      "bullet_points": [
        "1 hora de consultoria individual comigo",
        "Análise completa do seu negócio",
        "Plano de ação personalizado",
        "Suporte via WhatsApp por 30 dias"
      ],
      "original_price": 497.00,
      "upsell_price": 297.00,
      "timer_minutes": 15,
      "conversion_rate": 18.7,
      "total_conversions": 89,
      "is_active": true
    },
    {
      "id": "upsell-789",
      "name": "Upsell 2: Masterclass Bonus",
      "trigger_product_id": "prod-123",
      "upsell_product_id": "prod-101",
      "sequence_order": 2,
      "template": "testimonial",
      "conversion_rate": 12.3,
      "is_active": true
    }
  ]
}
```

#### `POST /api/funnel/upsells`
**Descrição**: Cria novo upsell
**Acesso**: Producer, Admin

```javascript
// Payload de exemplo
{
  "name": "Upsell 1: Consultoria Individual",
  "trigger_product_id": "prod-123",
  "upsell_product_id": "prod-789",
  "sequence_order": 1,
  "template": "video", // simple, video, testimonial, scarcity
  "headline": "🚀 Acelere Seus Resultados com Consultoria 1:1",
  "subheadline": "Apenas para os primeiros 20 alunos",
  "description": "Transforme seu aprendizado em resultados concretos com uma consultoria personalizada",
  "video_url": "https://exemplo.com/video-upsell.mp4",
  "image_url": "https://exemplo.com/consultoria.jpg",
  "bullet_points": [
    "1 hora de consultoria individual comigo",
    "Análise completa do seu negócio",
    "Plano de ação personalizado",
    "Suporte via WhatsApp por 30 dias"
  ],
  "testimonials": [
    {
      "name": "Maria Silva",
      "photo": "https://exemplo.com/maria.jpg",
      "text": "A consultoria mudou completamente meu negócio!",
      "rating": 5
    }
  ],
  "original_price": 497.00,
  "upsell_price": 297.00,
  "discount_percentage": 40,
  "timer_minutes": 15,
  "scarcity_settings": {
    "show_countdown": true,
    "show_stock": true,
    "stock_quantity": 20,
    "urgency_messages": [
      "Apenas 20 vagas disponíveis!",
      "Últimas vagas!"
    ]
  },
  "design_settings": {
    "background_color": "#1a1a1a",
    "text_color": "#ffffff",
    "button_color": "#ff6b35",
    "accent_color": "#ffc107"
  }
}
```

**Templates Disponíveis**:
- `simple`: Layout simples com texto e botão
- `video`: Página com vídeo de apresentação
- `testimonial`: Foco em depoimentos e provas sociais
- `scarcity`: Ênfase em urgência e escassez

### 3. Back Redirects

#### `POST /api/funnel/back-redirects`
**Descrição**: Cria estratégia de recuperação de carrinho abandonado
**Acesso**: Producer, Admin

```javascript
// Payload de exemplo
{
  "product_id": "prod-123",
  "trigger_event": "cart_abandon", // page_exit, cart_abandon, payment_fail
  "redirect_type": "discount_offer",
  "redirect_url": "https://exemplo.com/oferta-especial",
  "discount_settings": {
    "type": "percentage", // percentage, fixed_amount
    "value": 20,
    "max_discount": 100.00,
    "expires_hours": 24
  },
  "popup_settings": {
    "show_popup": true,
    "popup_delay": 3000,
    "popup_title": "🎯 Oferta Especial!",
    "popup_message": "Ganhe 20% de desconto se finalizar sua compra agora!",
    "popup_button_text": "Quero o Desconto",
    "popup_design": {
      "background_color": "#ffffff",
      "border_color": "#ff6b35",
      "text_color": "#333333",
      "button_color": "#28a745"
    }
  },
  "email_settings": {
    "send_email": true,
    "email_delay_minutes": 30,
    "email_template": "cart_recovery",
    "email_subject": "Você esqueceu algo no seu carrinho",
    "followup_emails": [
      {
        "delay_hours": 24,
        "subject": "Última chance: 20% OFF",
        "discount_increase": 5
      }
    ]
  },
  "sms_settings": {
    "send_sms": false,
    "sms_delay_minutes": 60,
    "sms_message": "Finalize sua compra e ganhe 20% OFF!"
  },
  "tracking_settings": {
    "track_page_exit": true,
    "track_time_on_page": true,
    "minimum_time_seconds": 30,
    "track_scroll_percentage": true,
    "minimum_scroll_percentage": 50
  }
}
```

**Eventos Gatilho**:
- `page_exit`: Usuário tenta sair da página
- `cart_abandon`: Carrinho abandonado por X tempo
- `payment_fail`: Falha no pagamento

### 4. Tracking de Conversões

#### `POST /api/funnel/track-conversion`
**Descrição**: Registra conversão do funil (endpoint público)
**Acesso**: Público (com validação)

```javascript
// Payload de exemplo
{
  "type": "order_bump", // order_bump, upsell, back_redirect
  "itemId": "bump-123",
  "orderId": "order-789",
  "customerData": {
    "customer_id": "cust-456",
    "email": "cliente@exemplo.com",
    "utm_source": "facebook",
    "utm_medium": "cpc",
    "utm_campaign": "campanha-2024",
    "conversion_value": 47.00,
    "original_value": 97.00,
    "session_data": {
      "ip_address": "192.168.1.1",
      "user_agent": "Mozilla/5.0...",
      "referrer": "https://facebook.com"
    }
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Conversão registrada com sucesso",
  "data": {
    "conversion_id": "conv-123",
    "type": "order_bump",
    "item_id": "bump-123",
    "order_id": "order-789",
    "conversion_value": 47.00,
    "timestamp": "2024-01-01T15:30:00Z"
  }
}
```

## Funcionalidades Avançadas

### 1. Segmentação Inteligente
```javascript
// Order bump com segmentação
{
  "triggers": {
    "show_for_all": false,
    "conditions": [
      {
        "type": "cart_value",
        "operator": "gte",
        "value": 100
      },
      {
        "type": "utm_source",
        "operator": "in",
        "value": ["facebook", "google"]
      },
      {
        "type": "customer_type",
        "operator": "equals",
        "value": "new"
      }
    ]
  }
}
```

### 2. A/B Testing
```javascript
// Configuração de teste A/B
{
  "ab_test": {
    "enabled": true,
    "test_name": "Headline Test",
    "variants": [
      {
        "name": "Variant A",
        "weight": 50,
        "headline": "🚀 Acelere Seus Resultados",
        "button_text": "Quero Acelerar"
      },
      {
        "name": "Variant B",
        "weight": 50,
        "headline": "🎯 Dobrar Seus Resultados",
        "button_text": "Quero Dobrar"
      }
    ]
  }
}
```

### 3. Analytics Avançadas
```javascript
// Métricas detalhadas
{
  "analytics": {
    "views": 1500,
    "conversions": 285,
    "conversion_rate": 19.0,
    "revenue": 13395.00,
    "average_order_value": 47.00,
    "roi": 340.5,
    "by_source": {
      "facebook": { "views": 800, "conversions": 152 },
      "google": { "views": 500, "conversions": 95 },
      "direct": { "views": 200, "conversions": 38 }
    },
    "by_device": {
      "mobile": { "views": 900, "conversions": 171 },
      "desktop": { "views": 600, "conversions": 114 }
    }
  }
}
```

### 4. Automações Avançadas
```javascript
// Sequência automatizada
{
  "automation_rules": [
    {
      "condition": "order_bump_accepted",
      "action": "show_upsell",
      "upsell_id": "upsell-789"
    },
    {
      "condition": "upsell_declined",
      "action": "show_downsell",
      "downsell_id": "downsell-456"
    },
    {
      "condition": "payment_failed",
      "action": "trigger_back_redirect",
      "redirect_id": "redirect-123"
    }
  ]
}
```

## Integração Frontend

### JavaScript SDK
```javascript
// Tracking de eventos
class FunnelTracker {
  constructor(apiUrl, productId) {
    this.apiUrl = apiUrl;
    this.productId = productId;
  }
  
  // Rastrear visualização de order bump
  trackOrderBumpView(bumpId) {
    this.sendEvent('order_bump_view', { bumpId });
  }
  
  // Rastrear conversão
  trackConversion(type, itemId, orderId, value) {
    fetch(`${this.apiUrl}/api/funnel/track-conversion`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        type,
        itemId,
        orderId,
        customerData: {
          conversion_value: value,
          session_data: this.getSessionData()
        }
      })
    });
  }
  
  getSessionData() {
    return {
      ip_address: this.getClientIP(),
      user_agent: navigator.userAgent,
      referrer: document.referrer,
      utm_params: this.getUTMParams()
    };
  }
}
```

### React Component Example
```jsx
import React, { useState, useEffect } from 'react';

const OrderBumpComponent = ({ bump, onAccept, onDecline }) => {
  const [isChecked, setIsChecked] = useState(bump.is_default_checked);
  
  useEffect(() => {
    // Rastrear visualização
    window.funnelTracker?.trackOrderBumpView(bump.id);
  }, [bump.id]);
  
  const handleToggle = () => {
    setIsChecked(!isChecked);
    if (!isChecked) {
      onAccept(bump);
    } else {
      onDecline(bump);
    }
  };
  
  return (
    <div className={`order-bump ${bump.position}`}>
      <div className="order-bump-content">
        <img src={bump.image_url} alt={bump.name} />
        <div className="order-bump-text">
          <h3>{bump.title}</h3>
          <p>{bump.description}</p>
          <div className="price">
            <span className="original-price">R$ {bump.original_price}</span>
            <span className="bump-price">R$ {bump.bump_price}</span>
          </div>
        </div>
        <label className="order-bump-checkbox">
          <input
            type="checkbox"
            checked={isChecked}
            onChange={handleToggle}
          />
          Adicionar ao pedido
        </label>
      </div>
    </div>
  );
};
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Dados inválidos ou campos obrigatórios faltando |
| 401 | Token de autenticação inválido |
| 403 | Sem permissão para gerenciar funil |
| 404 | Item do funil não encontrado |
| 500 | Erro interno do servidor |

## Considerações de Performance

1. **Lazy Loading**: Carregamento sob demanda de upsells
2. **Caching**: Cache de configurações de funil
3. **CDN**: Imagens e vídeos via CDN
4. **Compressão**: Assets otimizados
5. **Analytics**: Batch processing para métricas

## Segurança

1. **Rate Limiting**: Proteção contra spam
2. **Validation**: Sanitização de dados
3. **CORS**: Configuração adequada
4. **Tokens**: Autenticação obrigatória para gestão
5. **Ownership**: Verificação de propriedade

## Métricas de Sucesso

### KPIs Principais
- **Taxa de Conversão de Order Bump**: % de clientes que aceitam
- **Sequência de Upsell**: Taxa de conversão por etapa
- **Recuperação de Carrinho**: % de carrinhos abandonados recuperados
- **Valor Médio do Pedido**: Impacto do funil no AOV
- **ROI do Funil**: Retorno sobre investimento

### Benchmarks
- Order Bump: 15-30% de conversão
- Upsell 1: 10-25% de conversão
- Upsell 2: 5-15% de conversão
- Recuperação de Carrinho: 20-40%

Este sistema de funil oferece uma solução completa para maximizar conversões e aumentar o valor do carrinho através de estratégias comprovadas de marketing digital.
