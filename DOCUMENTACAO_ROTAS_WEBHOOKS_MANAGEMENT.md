# Documentação API - Gerenciamento de Webhooks

## Visão Geral
Sistema completo para criação, gerenciamento e monitoramento de webhooks, permitindo integração em tempo real com sistemas externos.

## Endpoints Disponíveis

### 1. Listar Webhooks
**GET** `/api/webhooks`

**Autenticação:** Requerida

**Descrição:** Lista todos os webhooks do usuário com paginação.

**Query Parameters:**
- `page` (number, opcional): Página atual (padrão: 1)
- `limit` (number, opcional): Itens por página (padrão: 20)
- `active` (boolean, opcional): Filtrar por status ativo

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "webhook_123456789",
      "url": "https://meusite.com/webhook/checkout",
      "events": [
        "order.created",
        "payment.confirmed",
        "order.completed"
      ],
      "active": true,
      "created_at": "2024-01-15T10:30:00Z",
      "last_triggered": "2024-01-20T14:45:00Z",
      "failures": 0,
      "has_secret": true
    },
    {
      "id": "webhook_987654321",
      "url": "https://sistema.empresa.com/hooks/vendas",
      "events": [
        "order.created",
        "order.cancelled"
      ],
      "active": false,
      "created_at": "2024-01-10T08:15:00Z",
      "last_triggered": null,
      "failures": 12,
      "has_secret": false
    }
  ],
  "pagination": {
    "current_page": 1,
    "total_pages": 3,
    "total_items": 45,
    "items_per_page": 20
  }
}
```

### 2. Criar Webhook
**POST** `/api/webhooks`

**Autenticação:** Requerida

**Rate Limit:** 50 requests por 15 minutos

**Descrição:** Cria um novo webhook para receber eventos.

**Body da Requisição:**
```json
{
  "url": "https://meusite.com/webhook/checkout",
  "events": [
    "order.created",
    "payment.confirmed",
    "order.completed"
  ],
  "secret": "meu_secret_personalizado"
}
```

**Parâmetros:**
- `url` (string, obrigatório): URL de destino do webhook (http/https)
- `events` (array, obrigatório): Lista de eventos para escutar
- `secret` (string, opcional): Secret para assinatura (gerado automaticamente se não fornecido)

**Resposta de Sucesso (201):**
```json
{
  "success": true,
  "message": "Webhook criado com sucesso",
  "data": {
    "id": "webhook_new123456",
    "url": "https://meusite.com/webhook/checkout",
    "events": [
      "order.created",
      "payment.confirmed",
      "order.completed"
    ],
    "active": true,
    "secret": "abc123def456ghi789jkl012mno345pqr678stu901vwx234yz567",
    "created_at": "2024-01-21T16:20:00Z"
  }
}
```

**Validações:**
- URL deve ser válida (http/https)
- Máximo 10 webhooks por usuário
- Eventos devem estar na lista de eventos disponíveis
- URL não pode ser duplicada para o mesmo usuário

**Erros Possíveis:**
- `400` - URL obrigatória ou inválida
- `400` - Eventos obrigatórios ou inválidos
- `400` - Limite máximo de 10 webhooks atingido
- `400` - Webhook com esta URL já existe
- `429` - Rate limit excedido

### 3. Obter Webhook Específico
**GET** `/api/webhooks/:id`

**Autenticação:** Requerida

**Descrição:** Retorna detalhes de um webhook específico incluindo logs recentes.

**Parâmetros de URL:**
- `id` (string): ID do webhook

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": {
    "id": "webhook_123456789",
    "url": "https://meusite.com/webhook/checkout",
    "events": [
      "order.created",
      "payment.confirmed"
    ],
    "active": true,
    "created_at": "2024-01-15T10:30:00Z",
    "last_triggered": "2024-01-20T14:45:00Z",
    "failures": 0,
    "has_secret": true,
    "recent_logs": [
      {
        "id": "log_abc123",
        "event_type": "order.created",
        "status": "success",
        "response_status": 200,
        "created_at": "2024-01-20T14:45:00Z",
        "error_message": null
      },
      {
        "id": "log_def456",
        "event_type": "payment.confirmed",
        "status": "failed",
        "response_status": 500,
        "created_at": "2024-01-20T12:30:00Z",
        "error_message": "HTTP 500 Internal Server Error"
      }
    ]
  }
}
```

### 4. Atualizar Webhook
**PUT** `/api/webhooks/:id`

**Autenticação:** Requerida

**Rate Limit:** 50 requests por 15 minutos

**Descrição:** Atualiza configurações de um webhook existente.

**Body da Requisição:**
```json
{
  "url": "https://novo-site.com/webhook/vendas",
  "events": [
    "order.created",
    "order.updated",
    "payment.confirmed"
  ],
  "active": true,
  "secret": "novo_secret_123456"
}
```

**Parâmetros (todos opcionais):**
- `url` (string): Nova URL do webhook
- `events` (array): Nova lista de eventos
- `active` (boolean): Ativar/desativar webhook
- `secret` (string): Novo secret para assinatura

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Webhook atualizado com sucesso",
  "data": {
    "id": "webhook_123456789",
    "url": "https://novo-site.com/webhook/vendas",
    "events": [
      "order.created",
      "order.updated",
      "payment.confirmed"
    ],
    "active": true,
    "has_secret": true,
    "updated_at": "2024-01-21T16:30:00Z"
  }
}
```

### 5. Remover Webhook
**DELETE** `/api/webhooks/:id`

**Autenticação:** Requerida

**Descrição:** Remove um webhook e todos os seus logs associados.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Webhook removido com sucesso"
}
```

### 6. Testar Webhook
**POST** `/api/webhooks/:id/test`

**Autenticação:** Requerida

**Descrição:** Envia um evento de teste para verificar se o webhook está funcionando.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Webhook testado com sucesso",
  "data": {
    "response_status": 200,
    "response_time": 245,
    "error": null
  }
}
```

**Resposta de Falha (200):**
```json
{
  "success": false,
  "message": "Falha no teste do webhook",
  "data": {
    "response_status": 500,
    "response_time": 30000,
    "error": "Connection timeout"
  }
}
```

**Payload de Teste Enviado:**
```json
{
  "event": "webhook.test",
  "timestamp": "2024-01-21T16:35:00Z",
  "data": {
    "message": "Este é um evento de teste do webhook",
    "webhook_id": "webhook_123456789",
    "test": true
  }
}
```

### 7. Logs do Webhook
**GET** `/api/webhooks/:id/logs`

**Autenticação:** Requerida

**Descrição:** Lista logs detalhados de um webhook específico.

**Query Parameters:**
- `page` (number, opcional): Página atual (padrão: 1)
- `limit` (number, opcional): Itens por página (padrão: 50)
- `status` (string, opcional): Filtrar por status (success/failed)

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "log_abc123def",
      "event_type": "order.created",
      "status": "success",
      "response_status": 200,
      "response_time": 234,
      "error_message": null,
      "created_at": "2024-01-20T14:45:00Z",
      "payload": {
        "event": "order.created",
        "timestamp": "2024-01-20T14:45:00Z",
        "data": {
          "order_id": "order_789",
          "customer_email": "cliente@email.com",
          "total": 199.90
        }
      }
    },
    {
      "id": "log_456ghi789",
      "event_type": "payment.confirmed",
      "status": "failed",
      "response_status": 404,
      "response_time": 1205,
      "error_message": "HTTP 404 Not Found",
      "created_at": "2024-01-20T12:30:00Z",
      "payload": {
        "event": "payment.confirmed",
        "timestamp": "2024-01-20T12:30:00Z",
        "data": {
          "payment_id": "pay_456",
          "order_id": "order_123",
          "amount": 299.90
        }
      }
    }
  ],
  "pagination": {
    "current_page": 1,
    "total_pages": 8,
    "total_items": 387,
    "items_per_page": 50
  }
}
```

### 8. Eventos Disponíveis
**GET** `/api/webhooks/events/available`

**Autenticação:** Requerida

**Descrição:** Lista todos os eventos disponíveis para webhooks.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "data": [
    {
      "name": "order.created",
      "description": "Disparado quando um novo pedido é criado"
    },
    {
      "name": "order.updated",
      "description": "Disparado quando um pedido é atualizado"
    },
    {
      "name": "order.completed",
      "description": "Disparado quando um pedido é concluído"
    },
    {
      "name": "order.cancelled",
      "description": "Disparado quando um pedido é cancelado"
    },
    {
      "name": "payment.confirmed",
      "description": "Disparado quando um pagamento é confirmado"
    },
    {
      "name": "payment.failed",
      "description": "Disparado quando um pagamento falha"
    },
    {
      "name": "product.created",
      "description": "Disparado quando um produto é criado"
    },
    {
      "name": "product.updated",
      "description": "Disparado quando um produto é atualizado"
    },
    {
      "name": "user.registered",
      "description": "Disparado quando um usuário se registra"
    },
    {
      "name": "checkout.abandoned",
      "description": "Disparado quando um checkout é abandonado"
    },
    {
      "name": "checkout.recovered",
      "description": "Disparado quando um checkout abandonado é recuperado"
    }
  ]
}
```

## Estrutura de Eventos

### Formato Padrão do Payload
```json
{
  "event": "order.created",
  "timestamp": "2024-01-21T16:40:00Z",
  "data": {
    // Dados específicos do evento
  }
}
```

### Eventos de Pedido

#### order.created
```json
{
  "event": "order.created",
  "timestamp": "2024-01-21T16:40:00Z",
  "data": {
    "order_id": "order_123456789",
    "customer": {
      "email": "cliente@email.com",
      "name": "João Silva",
      "phone": "+5511999999999"
    },
    "product": {
      "id": "prod_987654321",
      "name": "Curso de Marketing Digital",
      "price": 299.90
    },
    "payment": {
      "method": "credit_card",
      "status": "pending"
    },
    "total": 299.90,
    "created_at": "2024-01-21T16:40:00Z"
  }
}
```

#### order.completed
```json
{
  "event": "order.completed",
  "timestamp": "2024-01-21T16:45:00Z",
  "data": {
    "order_id": "order_123456789",
    "customer": {
      "email": "cliente@email.com",
      "access_granted": true
    },
    "product": {
      "id": "prod_987654321",
      "access_url": "https://app.checkout.pro/course/987654321"
    },
    "payment": {
      "confirmed_at": "2024-01-21T16:45:00Z",
      "transaction_id": "txn_abc123def456"
    },
    "total": 299.90
  }
}
```

### Eventos de Pagamento

#### payment.confirmed
```json
{
  "event": "payment.confirmed",
  "timestamp": "2024-01-21T16:45:00Z",
  "data": {
    "payment_id": "pay_123456789",
    "order_id": "order_987654321",
    "method": "credit_card",
    "amount": 299.90,
    "gateway": "stripe",
    "transaction_id": "txn_abc123def456",
    "confirmed_at": "2024-01-21T16:45:00Z"
  }
}
```

#### payment.failed
```json
{
  "event": "payment.failed",
  "timestamp": "2024-01-21T16:50:00Z",
  "data": {
    "payment_id": "pay_987654321",
    "order_id": "order_123456789",
    "method": "credit_card",
    "amount": 299.90,
    "failure_reason": "insufficient_funds",
    "gateway_error": "Your card was declined",
    "failed_at": "2024-01-21T16:50:00Z"
  }
}
```

## Assinatura de Webhooks

### Geração da Assinatura
```javascript
const crypto = require('crypto');

const generateSignature = (payload, secret) => {
  const signature = crypto
    .createHmac('sha256', secret)
    .update(JSON.stringify(payload))
    .digest('hex');
  
  return `sha256=${signature}`;
};
```

### Headers de Segurança
```javascript
// Headers enviados com cada webhook
{
  'Content-Type': 'application/json',
  'User-Agent': 'CheckoutPro-Webhook/1.0',
  'X-CheckoutPro-Signature': 'sha256=a1b2c3d4e5f6...' // Se webhook tem secret
}
```

### Verificação de Assinatura (Lado Receptor)
```javascript
// Node.js - Express
app.post('/webhook', (req, res) => {
  const signature = req.headers['x-checkoutpro-signature'];
  const payload = JSON.stringify(req.body);
  const secret = 'seu_secret_do_webhook';
  
  if (signature) {
    const expectedSignature = `sha256=${crypto
      .createHmac('sha256', secret)
      .update(payload)
      .digest('hex')}`;
    
    if (signature !== expectedSignature) {
      return res.status(401).json({ error: 'Invalid signature' });
    }
  }
  
  // Processar webhook
  console.log('Evento recebido:', req.body.event);
  res.status(200).json({ received: true });
});
```

## Sistema de Retry e Falhas

### Lógica de Retry
- **Timeout**: 30 segundos por tentativa
- **Retry Automático**: Não implementado (logs são mantidos)
- **Falhas Consecutivas**: Webhook desativado após 10 falhas
- **Reset de Falhas**: Contador zerado em sucesso

### Monitoramento de Saúde
```javascript
// Verificar saúde dos webhooks
const checkWebhookHealth = async (userId) => {
  const webhooks = await Webhook.findAll({
    where: { user_id: userId },
    attributes: ['id', 'url', 'active', 'failures', 'last_triggered']
  });
  
  return webhooks.map(webhook => ({
    id: webhook.id,
    url: webhook.url,
    status: webhook.active ? 'active' : 'inactive',
    health: webhook.failures < 5 ? 'healthy' : 'degraded',
    last_activity: webhook.last_triggered
  }));
};
```

## Implementação do Cliente

### Webhook Receiver Básico
```javascript
// Express.js
const express = require('express');
const crypto = require('crypto');
const app = express();

app.use(express.json());

// Middleware para verificar assinatura
const verifyWebhookSignature = (secret) => {
  return (req, res, next) => {
    const signature = req.headers['x-checkoutpro-signature'];
    
    if (signature && secret) {
      const payload = JSON.stringify(req.body);
      const expectedSignature = `sha256=${crypto
        .createHmac('sha256', secret)
        .update(payload)
        .digest('hex')}`;
      
      if (signature !== expectedSignature) {
        return res.status(401).json({ error: 'Invalid signature' });
      }
    }
    
    next();
  };
};

// Endpoint do webhook
app.post('/webhook/checkout', verifyWebhookSignature('seu_secret'), (req, res) => {
  const { event, data } = req.body;
  
  switch (event) {
    case 'order.created':
      console.log(`Novo pedido: ${data.order_id}`);
      // Lógica para novo pedido
      break;
      
    case 'payment.confirmed':
      console.log(`Pagamento confirmado: ${data.payment_id}`);
      // Liberar acesso ao produto
      break;
      
    case 'order.cancelled':
      console.log(`Pedido cancelado: ${data.order_id}`);
      // Lógica para cancelamento
      break;
      
    default:
      console.log(`Evento não tratado: ${event}`);
  }
  
  // Sempre responder com 200 para evitar reenvios
  res.status(200).json({ received: true });
});

app.listen(3000, () => {
  console.log('Webhook receiver rodando na porta 3000');
});
```

### Cliente para Gerenciamento
```javascript
class WebhookClient {
  constructor(apiUrl, token) {
    this.apiUrl = apiUrl;
    this.token = token;
  }
  
  async createWebhook(url, events, secret = null) {
    const response = await fetch(`${this.apiUrl}/api/webhooks`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ url, events, secret })
    });
    
    return response.json();
  }
  
  async listWebhooks(page = 1, limit = 20) {
    const response = await fetch(
      `${this.apiUrl}/api/webhooks?page=${page}&limit=${limit}`,
      {
        headers: { 'Authorization': `Bearer ${this.token}` }
      }
    );
    
    return response.json();
  }
  
  async testWebhook(webhookId) {
    const response = await fetch(
      `${this.apiUrl}/api/webhooks/${webhookId}/test`,
      {
        method: 'POST',
        headers: { 'Authorization': `Bearer ${this.token}` }
      }
    );
    
    return response.json();
  }
  
  async getWebhookLogs(webhookId, page = 1) {
    const response = await fetch(
      `${this.apiUrl}/api/webhooks/${webhookId}/logs?page=${page}`,
      {
        headers: { 'Authorization': `Bearer ${this.token}` }
      }
    );
    
    return response.json();
  }
}

// Uso
const client = new WebhookClient('https://api.checkout.pro', 'seu_token');

// Criar webhook
await client.createWebhook(
  'https://meusite.com/webhook',
  ['order.created', 'payment.confirmed']
);

// Listar webhooks
const webhooks = await client.listWebhooks();

// Testar webhook
await client.testWebhook('webhook_123456789');
```

## Boas Práticas

### Para Desenvolvedores
1. **Idempotência**: Trate eventos duplicados
2. **Timeout**: Configure timeout apropriado (30s+)
3. **Logs**: Registre todos os eventos recebidos
4. **Status 200**: Sempre responda com sucesso
5. **Processamento Assíncrono**: Use filas para processamento pesado

### Para Segurança
1. **Verificar Assinatura**: Sempre valide quando disponível
2. **HTTPS**: Use apenas URLs HTTPS em produção
3. **Rate Limiting**: Implemente proteção no receptor
4. **Validação**: Valide estrutura dos dados recebidos

### Para Monitoramento
1. **Health Checks**: Monitore falhas consecutivas
2. **Alertas**: Configure alertas para webhooks inativos
3. **Logs Detalhados**: Mantenha logs para auditoria
4. **Métricas**: Acompanhe taxa de sucesso/falha

## Limitações e Considerações

1. **Máximo de Webhooks**: 10 por usuário
2. **Timeout**: 30 segundos por tentativa
3. **Retry**: Não há retry automático
4. **Payload Size**: Sem limite específico documentado
5. **Rate Limiting**: 50 requests por 15 minutos para operações
6. **Desativação**: Automática após 10 falhas consecutivas
7. **URL Única**: Não permite URLs duplicadas por usuário
