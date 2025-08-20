# Documentação - Sistema de Tracking (routes/tracking.js)

## Visão Geral
Sistema completo de tracking e pixels de rastreamento que permite aos produtores configurar e gerenciar pixels de diferentes provedores (Google Analytics, Facebook, TikTok, Kwai) para monitorar eventos em seus produtos.

## Configuração
- **Rate Limiting**: 1000 requests por minuto por IP
- **Autenticação**: JWT token required para rotas administrativas
- **Autorização**: Produtores e Admins para gerenciamento de pixels
- **Logs**: Sistema de auditoria integrado

## Rotas Implementadas

### 1. Registro de Eventos (Público)

#### `POST /api/tracking/events`
**Descrição**: Registra eventos de tracking de forma pública
**Acesso**: Público (com rate limiting)
**Rate Limit**: 1000 requests/minuto

```javascript
// Payload de exemplo
{
  "provider": "googleAnalytics", // googleAnalytics, facebookPixel, tiktokPixel, kwaiPixel
  "event": "page_view",
  "data": {
    "product_id": "123",
    "user_id": "456",
    "value": 99.90,
    "currency": "BRL",
    "utm_source": "google",
    "utm_medium": "cpc",
    "utm_campaign": "campanha1"
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Evento registrado com sucesso"
}
```

**Eventos Suportados**:
- `page_view`: Visualização de página
- `product_view`: Visualização de produto
- `add_to_cart`: Adicionar ao carrinho
- `begin_checkout`: Iniciar checkout
- `purchase`: Compra realizada
- `view_item`: Visualizar item
- `conversion`: Conversão genérica
- `complete_registration`: Registro completo

### 2. Analytics de Produto

#### `GET /api/tracking/analytics/:productId`
**Descrição**: Retorna analytics detalhadas de um produto específico
**Acesso**: Privado (token required)

**Response Success**:
```json
{
  "success": true,
  "data": {
    "views": 1250,
    "conversions": 45,
    "revenue": 4485.50,
    "conversion_rate": 3.6,
    "sources": [
      {
        "source": "google",
        "count": 850
      },
      {
        "source": "facebook",
        "count": 300
      },
      {
        "source": "direct",
        "count": 100
      }
    ],
    "events_summary": {
      "total_events": 2500,
      "last_30_days": 750
    }
  }
}
```

**Métricas Calculadas**:
- Total de visualizações
- Total de conversões
- Receita total
- Taxa de conversão
- Fontes de tráfego
- Resumo de eventos

### 3. Gerenciamento de Pixels

#### `GET /api/tracking/pixels`
**Descrição**: Lista todos os pixels do usuário
**Acesso**: Producer, Admin

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "id": "pixel-123",
      "name": "Google Analytics - Loja Principal",
      "provider": "google",
      "pixel_id": "GA_MEASUREMENT_ID",
      "is_active": true,
      "created_at": "2024-01-01T10:00:00Z"
    }
  ]
}
```

#### `POST /api/tracking/pixels`
**Descrição**: Cria novo pixel de rastreamento
**Acesso**: Producer, Admin

```javascript
// Payload
{
  "name": "Facebook Pixel - Campanha Verão",
  "provider": "facebook", // google, kwai, tiktok, facebook, custom
  "pixel_id": "FB_PIXEL_ID",
  "config": {
    "events": ["page_view", "purchase"],
    "auto_track": true
  }
}
```

**Provedores Suportados**:
- `google`: Google Analytics
- `facebook`: Facebook Pixel
- `tiktok`: TikTok Pixel
- `kwai`: Kwai Pixel
- `custom`: Pixel customizado

#### `PUT /api/tracking/pixels/:id`
**Descrição**: Atualiza configurações do pixel
**Acesso**: Producer, Admin

```javascript
// Payload de exemplo
{
  "name": "Nome Atualizado",
  "is_active": false,
  "config": {
    "events": ["page_view", "purchase", "add_to_cart"]
  }
}
```

#### `DELETE /api/tracking/pixels/:id`
**Descrição**: Desativa pixel (soft delete)
**Acesso**: Producer, Admin

**Response Success**:
```json
{
  "success": true,
  "message": "Pixel desativado com sucesso"
}
```

### 4. Gestão de Produtos e Pixels

#### `POST /api/tracking/pixels/:id/attach-product`
**Descrição**: Anexa pixel a um produto específico
**Acesso**: Producer, Admin

```javascript
// Payload
{
  "productId": "product-456",
  "events": ["page_view", "purchase", "add_to_cart"]
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Pixel anexado ao produto com sucesso",
  "data": {
    "pixel_id": "pixel-123",
    "product_id": "product-456",
    "events": ["page_view", "purchase", "add_to_cart"]
  }
}
```

### 5. Eventos do Pixel

#### `GET /api/tracking/pixels/:id/events`
**Descrição**: Lista eventos registrados pelo pixel
**Acesso**: Producer, Admin

**Query Parameters**:
- `eventType`: Filtrar por tipo de evento
- `status`: Filtrar por status (success, failed, pending)
- `startDate`: Data inicial (ISO format)
- `endDate`: Data final (ISO format)
- `limit`: Limite de resultados (padrão: 100)

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "id": "event-789",
      "event_type": "purchase",
      "event_data": {
        "value": 99.90,
        "currency": "BRL",
        "product_id": "123"
      },
      "status": "success",
      "ip_address": "192.168.1.1",
      "user_agent": "Mozilla/5.0...",
      "created_at": "2024-01-01T15:30:00Z"
    }
  ],
  "total": 1
}
```

#### `POST /api/tracking/pixels/:id/retry-failed`
**Descrição**: Reprocessa eventos que falharam
**Acesso**: Producer, Admin

**Response Success**:
```json
{
  "success": true,
  "message": "Retry de eventos executado",
  "results": [
    {
      "event_id": "event-456",
      "status": "success",
      "retry_count": 1
    }
  ]
}
```

### 6. Envio de Evento (Frontend)

#### `POST /api/tracking/event`
**Descrição**: Endpoint público para JavaScript do frontend enviar eventos
**Acesso**: Público

```javascript
// Payload
{
  "pixelId": "pixel-123",
  "eventType": "purchase",
  "eventData": {
    "value": 99.90,
    "currency": "BRL",
    "product_id": "123",
    "user_id": "456"
  },
  "sessionId": "session-789"
}
```

**Informações Coletadas Automaticamente**:
- IP Address
- User Agent
- Session ID
- User ID (se autenticado)

### 7. Provedores Disponíveis

#### `GET /api/tracking/providers`
**Descrição**: Lista provedores de pixel disponíveis
**Acesso**: Producer, Admin

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "id": "google",
      "name": "Google Analytics",
      "description": "Google Analytics 4",
      "events": ["page_view", "purchase", "add_to_cart"]
    },
    {
      "id": "facebook",
      "name": "Facebook Pixel",
      "description": "Meta Pixel para Facebook e Instagram",
      "events": ["PageView", "Purchase", "AddToCart"]
    }
  ]
}
```

## Segurança e Validações

### Rate Limiting
- **Limite**: 1000 requests por minuto por IP
- **Escopo**: Rotas de eventos públicos
- **Comportamento**: Retorna 429 quando excedido

### Autenticação
- **JWT Token**: Required para rotas administrativas
- **Middleware**: `authenticateToken`
- **Scope**: Todas as rotas exceto `/events` e `/event`

### Autorização
- **Níveis**: Producer, Admin
- **Middleware**: `requirePixelAccess`
- **Validação**: Ownership de pixels verificada

### Validações de Dados
1. **Providers**: Apenas providers suportados
2. **Event Types**: Eventos válidos por provider
3. **Product Ownership**: Verificação de propriedade
4. **Pixel Ownership**: Verificação de propriedade
5. **Required Fields**: Campos obrigatórios validados

## Funcionalidades Avançadas

### 1. Tracking Automático
- Coleta automática de IP e User Agent
- Session tracking
- User identification quando autenticado

### 2. Retry de Eventos
- Sistema de retry para eventos falhados
- Tracking de tentativas
- Status detalhado dos eventos

### 3. Analytics Integradas
- Cálculo automático de métricas
- Agrupamento por fonte de tráfego
- Períodos de análise configuráveis

### 4. Auditoria Completa
- Log de todas as ações
- Rastreamento de modificações
- Histórico de eventos

### 5. Soft Delete
- Pixels desativados ao invés de deletados
- Preservação de dados históricos
- Recuperação possível

## Modelos de Dados

### TrackingPixel
```javascript
{
  id: "string",
  user_id: "string",
  name: "string",
  provider: "string",
  pixel_id: "string",
  config: "object",
  is_active: "boolean",
  created_at: "datetime",
  updated_at: "datetime"
}
```

### TrackingEvent
```javascript
{
  id: "string",
  pixel_id: "string",
  product_id: "string",
  event_type: "string",
  event_data: "object",
  status: "string",
  ip_address: "string",
  user_agent: "string",
  session_id: "string",
  user_id: "string",
  created_at: "datetime"
}
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Dados inválidos ou campos obrigatórios faltando |
| 401 | Token de autenticação inválido |
| 403 | Sem permissão para acessar recurso |
| 404 | Pixel ou produto não encontrado |
| 429 | Rate limit excedido |
| 500 | Erro interno do servidor |

## Integração Frontend

### JavaScript Tracking
```javascript
// Exemplo de integração
async function trackEvent(eventType, data) {
  try {
    const response = await fetch('/api/tracking/event', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        pixelId: 'pixel-123',
        eventType: eventType,
        eventData: data,
        sessionId: getSessionId()
      })
    });
    
    const result = await response.json();
    console.log('Event tracked:', result);
  } catch (error) {
    console.error('Tracking error:', error);
  }
}

// Uso
trackEvent('purchase', {
  value: 99.90,
  currency: 'BRL',
  product_id: '123'
});
```

## Considerações de Performance

1. **Rate Limiting**: Protege contra spam e abuse
2. **Indexação**: Eventos indexados por pixel_id e created_at
3. **Paginação**: Implementada para listas grandes
4. **Async Processing**: Eventos processados de forma assíncrona
5. **Caching**: Providers e configurações em cache

## Monitoramento

### Métricas Importantes
- Taxa de eventos falhados
- Latência de processamento
- Volume de eventos por provider
- Taxa de retry bem-sucedidos

### Logs de Auditoria
- Criação/modificação de pixels
- Anexação de produtos
- Retry de eventos
- Alterações de configuração

Este sistema oferece uma solução completa para tracking e analytics, permitindo aos produtores monitorar efetivamente o comportamento dos usuários em seus produtos e otimizar suas estratégias de marketing.
