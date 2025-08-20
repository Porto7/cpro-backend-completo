# Documentação API - Sistema de Webhooks (Core)

## Visão Geral
Sistema principal de webhooks para gerenciamento de eventos e integrações em tempo real. Este é o sistema core de webhooks complementar ao sistema de gerenciamento avançado.

## Endpoints Disponíveis

### 1. Listar Webhooks do Usuário
**GET** `/api/webhooks`

**Autenticação:** Requerida

**Descrição:** Lista todos os webhooks do usuário autenticado.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "webhooks": [
    {
      "id": "webhook_123456789",
      "user_id": "user_987654321",
      "name": "Webhook Principal",
      "url": "https://meusite.com/webhook",
      "secret": "abc123def456...",
      "events": [
        "order.created",
        "sale.completed"
      ],
      "active": true,
      "stats": {
        "total_calls": 150,
        "successful_calls": 147,
        "failed_calls": 3,
        "last_success": "2024-01-21T16:30:00Z",
        "last_failure": "2024-01-20T10:15:00Z"
      },
      "retryCount": 3,
      "timeout": 30,
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-21T16:30:00Z"
    }
  ]
}
```

### 2. Criar Webhook
**POST** `/api/webhooks`

**Autenticação:** Requerida

**Descrição:** Cria um novo webhook para o usuário.

**Body da Requisição:**
```json
{
  "name": "Webhook de Vendas",
  "url": "https://meusite.com/api/webhook/vendas",
  "secret": "meu_secret_personalizado",
  "events": [
    "order.created",
    "sale.completed",
    "sale.cancelled"
  ]
}
```

**Parâmetros:**
- `name` (string, obrigatório): Nome identificador do webhook
- `url` (string, obrigatório): URL de destino
- `secret` (string, opcional): Secret para assinatura (gerado automaticamente se não fornecido)
- `events` (array, obrigatório): Lista de eventos para escutar

**Resposta de Sucesso (201):**
```json
{
  "success": true,
  "webhook": {
    "id": "webhook_new123456",
    "user_id": "user_987654321",
    "name": "Webhook de Vendas",
    "url": "https://meusite.com/api/webhook/vendas",
    "secret": "meu_secret_personalizado",
    "events": [
      "order.created",
      "sale.completed",
      "sale.cancelled"
    ],
    "active": true,
    "retryCount": 3,
    "timeout": 30,
    "created_at": "2024-01-21T17:00:00Z"
  },
  "message": "Webhook criado com sucesso"
}
```

### 3. Atualizar Webhook
**PUT** `/api/webhooks/:id`

**Autenticação:** Requerida

**Descrição:** Atualiza configurações de um webhook existente.

**Body da Requisição:**
```json
{
  "name": "Webhook Atualizado",
  "url": "https://novo-site.com/webhook",
  "secret": "novo_secret",
  "events": [
    "order.created",
    "order.updated",
    "sale.completed"
  ],
  "active": false
}
```

**Parâmetros (todos opcionais):**
- `name` (string): Novo nome do webhook
- `url` (string): Nova URL
- `secret` (string): Novo secret
- `events` (array): Nova lista de eventos
- `active` (boolean): Ativar/desativar webhook

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "webhook": {
    "id": "webhook_123456789",
    "name": "Webhook Atualizado",
    "url": "https://novo-site.com/webhook",
    "events": [
      "order.created",
      "order.updated",
      "sale.completed"
    ],
    "active": false,
    "updated_at": "2024-01-21T17:10:00Z"
  }
}
```

### 4. Deletar Webhook
**DELETE** `/api/webhooks/:id`

**Autenticação:** Requerida

**Descrição:** Remove permanentemente um webhook.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Webhook deletado com sucesso"
}
```

### 5. Testar Webhook (Simples)
**POST** `/api/webhooks/:id/test`

**Autenticação:** Requerida

**Descrição:** Envia um evento de teste simples para o webhook.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Webhook de teste enviado",
  "data": {
    "test": true,
    "message": "Webhook de teste",
    "timestamp": "2024-01-21T17:15:00Z"
  }
}
```

### 6. Logs do Webhook
**GET** `/api/webhooks/:id/logs`

**Autenticação:** Requerida

**Descrição:** Lista logs de execução de um webhook específico.

**Query Parameters:**
- `page` (number, opcional): Página atual (padrão: 1)
- `limit` (number, opcional): Itens por página (padrão: 50, máximo: 100)

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "logs": [
    {
      "id": "log_abc123",
      "webhook_id": "webhook_123456789",
      "event_type": "order.created",
      "payload": "{\"event\":\"order.created\",\"data\":{...}}",
      "response_status": 200,
      "response_body": "{\"received\":true}",
      "success": true,
      "error_message": null,
      "attempt_number": 1,
      "response_time": 245,
      "created_at": "2024-01-21T16:30:00Z"
    },
    {
      "id": "log_def456",
      "webhook_id": "webhook_123456789",
      "event_type": "sale.completed",
      "payload": "{\"event\":\"sale.completed\",\"data\":{...}}",
      "response_status": 500,
      "response_body": "Internal Server Error",
      "success": false,
      "error_message": "HTTP 500 response",
      "attempt_number": 2,
      "response_time": 5000,
      "created_at": "2024-01-21T15:45:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 50,
    "total": 150,
    "pages": 3
  }
}
```

### 7. Estatísticas Gerais
**GET** `/api/webhooks/stats`

**Autenticação:** Requerida

**Descrição:** Retorna estatísticas gerais dos webhooks do usuário.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "stats": {
    "total_webhooks": 5,
    "active_webhooks": 4,
    "calls_today": 45,
    "success_rate": 96.5
  }
}
```

### 8. Eventos Disponíveis (Público)
**GET** `/api/webhooks/events`

**Autenticação:** Não requerida

**Descrição:** Lista todos os eventos disponíveis para configuração em webhooks.

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "events": [
    {
      "value": "order.created",
      "label": "Order Created"
    },
    {
      "value": "order.updated",
      "label": "Order Updated"
    },
    {
      "value": "sale.completed",
      "label": "Sale Completed"
    },
    {
      "value": "sale.pending",
      "label": "Sale Pending"
    },
    {
      "value": "sale.cancelled",
      "label": "Sale Cancelled"
    },
    {
      "value": "customer.created",
      "label": "Customer Created"
    },
    {
      "value": "product.created",
      "label": "Product Created"
    },
    {
      "value": "product.updated",
      "label": "Product Updated"
    },
    {
      "value": "test.webhook",
      "label": "Test Webhook"
    }
  ]
}
```

### 9. Configurar Webhook Personalizado
**POST** `/api/webhooks/configure`

**Autenticação:** Requerida

**Descrição:** Endpoint avançado para configurar webhook com validações completas.

**Body da Requisição:**
```json
{
  "name": "Webhook CRM Integration",
  "url": "https://meu-crm.com/webhook/checkout",
  "events": [
    "customer.created",
    "sale.completed"
  ],
  "secret": "super_secret_123"
}
```

**Resposta de Sucesso (201):**
```json
{
  "success": true,
  "message": "Webhook configurado com sucesso",
  "data": {
    "id": "webhook_config123",
    "name": "Webhook CRM Integration",
    "url": "https://meu-crm.com/webhook/checkout",
    "events": [
      "customer.created",
      "sale.completed"
    ],
    "secret": "super_secret_123",
    "active": true
  }
}
```

**Validações Avançadas:**
- Verifica se eventos são válidos
- Valida formato da URL
- Retorna lista de eventos disponíveis em caso de erro

**Erros Possíveis:**
- `400` - Eventos obrigatórios ou inválidos
- `400` - URL inválida
- `400` - Eventos inválidos (retorna lista de válidos)

### 10. Webhook para Provedores de Pagamento
**POST** `/api/webhooks/payment/:provider`

**Autenticação:** Não requerida (público)

**Descrição:** Endpoint genérico para receber webhooks de provedores de pagamento externos.

**Parâmetros de URL:**
- `provider` (string): Nome do provedor (stripe, paypal, pagseguro, mercadopago)

**Provedores Suportados:**
- `stripe` - Stripe Payments
- `paypal` - PayPal
- `pagseguro` - PagSeguro
- `mercadopago` - Mercado Pago

**Exemplo Stripe:**
```bash
POST /api/webhooks/payment/stripe
```

**Body (Stripe):**
```json
{
  "id": "evt_1234567890",
  "object": "event",
  "type": "payment_intent.succeeded",
  "data": {
    "object": {
      "id": "pi_1234567890",
      "amount": 2000,
      "currency": "usd",
      "status": "succeeded"
    }
  }
}
```

**Resposta de Sucesso (200):**
```json
{
  "success": true,
  "message": "Webhook processado com sucesso"
}
```

## Sistema de Retry Automático

### Configuração de Retry
- **Tentativas Máximas**: 3 por padrão (configurável)
- **Backoff Exponencial**: 2^attempt segundos entre tentativas
- **Timeout**: 30 segundos por tentativa (configurável)

### Lógica de Retry
```javascript
// Sequência de tentativas
Tentativa 1: Imediata
Tentativa 2: Após 2 segundos
Tentativa 3: Após 4 segundos
```

### Logs de Tentativas
Cada tentativa gera um log separado com:
- Número da tentativa
- Status da resposta
- Tempo de resposta
- Mensagem de erro (se houver)

## Formato de Eventos Webhook

### Estrutura Padrão
```json
{
  "event": "order.created",
  "data": {
    // Dados específicos do evento
  },
  "timestamp": "2024-01-21T17:20:00Z",
  "webhook_id": "webhook_123456789"
}
```

### Assinatura HMAC
```javascript
// Header enviado
"X-Webhook-Signature": "sha256=a1b2c3d4e5f6..."

// Verificação
const signature = crypto
  .createHmac('sha256', webhook_secret)
  .update(payload_string, 'utf8')
  .digest('hex');

const expectedSignature = `sha256=${signature}`;
```

## Eventos Disponíveis

### Eventos de Pedido
- `order.created` - Novo pedido criado
- `order.updated` - Pedido atualizado

### Eventos de Venda
- `sale.completed` - Venda concluída com sucesso
- `sale.pending` - Venda pendente de confirmação
- `sale.cancelled` - Venda cancelada

### Eventos de Cliente
- `customer.created` - Novo cliente registrado

### Eventos de Produto
- `product.created` - Produto criado
- `product.updated` - Produto atualizado

### Eventos de Teste
- `test.webhook` - Evento para testes

## Função de Dispatch (Para Uso Interno)

### dispatchWebhook()
```javascript
import { dispatchWebhook } from './routes/webhooks.js';

// Usar em outras partes do sistema
await dispatchWebhook(
  userId,           // ID do usuário
  'order.created',  // Tipo do evento
  {                 // Dados do evento
    order_id: 'order_123',
    customer_email: 'cliente@email.com',
    total: 299.90
  }
);
```

### Uso Automático
```javascript
// Exemplo: Disparar webhook após criar pedido
const createOrder = async (orderData) => {
  // Criar pedido
  const order = await Order.create(orderData);
  
  // Disparar webhooks automaticamente
  await dispatchWebhook(orderData.user_id, 'order.created', {
    order_id: order.id,
    customer_email: order.customer_email,
    total: order.total,
    product_name: order.product_name
  });
  
  return order;
};
```

## Implementação no Cliente

### Receptor Básico
```javascript
const express = require('express');
const crypto = require('crypto');
const app = express();

app.use(express.json());

// Verificar assinatura
const verifySignature = (req, res, next) => {
  const signature = req.headers['x-webhook-signature'];
  const secret = 'seu_webhook_secret';
  
  if (signature && secret) {
    const payload = JSON.stringify(req.body);
    const expectedSignature = `sha256=${crypto
      .createHmac('sha256', secret)
      .update(payload, 'utf8')
      .digest('hex')}`;
    
    if (signature !== expectedSignature) {
      return res.status(401).json({ error: 'Assinatura inválida' });
    }
  }
  
  next();
};

// Processar webhook
app.post('/webhook', verifySignature, (req, res) => {
  const { event, data, timestamp } = req.body;
  
  console.log(`Evento recebido: ${event} em ${timestamp}`);
  
  switch (event) {
    case 'order.created':
      handleNewOrder(data);
      break;
    case 'sale.completed':
      handleSaleCompleted(data);
      break;
    default:
      console.log(`Evento não tratado: ${event}`);
  }
  
  res.json({ received: true });
});

const handleNewOrder = (data) => {
  console.log(`Novo pedido: ${data.order_id}`);
  // Sua lógica aqui
};

const handleSaleCompleted = (data) => {
  console.log(`Venda completa: ${data.order_id}`);
  // Liberar acesso, enviar email, etc.
};

app.listen(3000, () => {
  console.log('Webhook receiver rodando na porta 3000');
});
```

### Cliente de Gerenciamento
```javascript
class SimpleWebhookClient {
  constructor(apiUrl, token) {
    this.apiUrl = apiUrl;
    this.token = token;
  }
  
  async createWebhook(name, url, events, secret = null) {
    const response = await fetch(`${this.apiUrl}/api/webhooks`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ name, url, events, secret })
    });
    
    return response.json();
  }
  
  async listWebhooks() {
    const response = await fetch(`${this.apiUrl}/api/webhooks`, {
      headers: { 'Authorization': `Bearer ${this.token}` }
    });
    
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
  
  async getStats() {
    const response = await fetch(`${this.apiUrl}/api/webhooks/stats`, {
      headers: { 'Authorization': `Bearer ${this.token}` }
    });
    
    return response.json();
  }
}

// Uso
const client = new SimpleWebhookClient('https://api.checkout.pro', 'seu_token');

// Criar webhook
const webhook = await client.createWebhook(
  'Meu Webhook',
  'https://meusite.com/webhook',
  ['order.created', 'sale.completed']
);

// Listar webhooks
const webhooks = await client.listWebhooks();
console.log(`${webhooks.webhooks.length} webhooks encontrados`);

// Estatísticas
const stats = await client.getStats();
console.log(`Taxa de sucesso: ${stats.stats.success_rate}%`);
```

## Integração com Provedores

### Processamento por Provedor
O sistema inclui handlers específicos para diferentes provedores de pagamento:

```javascript
// Stripe
POST /api/webhooks/payment/stripe
// Processa eventos: payment_intent.succeeded, payment_intent.payment_failed

// PayPal  
POST /api/webhooks/payment/paypal
// Processa eventos específicos do PayPal

// PagSeguro
POST /api/webhooks/payment/pagseguro
// Processa notificações do PagSeguro

// Mercado Pago
POST /api/webhooks/payment/mercadopago
// Processa eventos do Mercado Pago
```

## Monitoramento e Logs

### Logs Detalhados
Cada execução de webhook gera logs com:
- Payload enviado
- Resposta recebida (primeiros 1000 caracteres)
- Status HTTP
- Tempo de resposta
- Número da tentativa
- Mensagens de erro

### Estatísticas Automáticas
O sistema mantém automaticamente:
- Total de chamadas
- Chamadas bem-sucedidas
- Chamadas falhadas
- Última execução com sucesso
- Última falha

## Observações Importantes

1. **Retry Automático**: Sistema implementa retry automático com backoff exponencial
2. **Logs Detalhados**: Todos os webhooks são logados para auditoria
3. **Assinatura HMAC**: Suporte a verificação de assinatura para segurança
4. **Provedores Externos**: Endpoints dedicados para webhooks de pagamento
5. **Eventos Públicos**: Lista de eventos disponível publicamente
6. **Timeouts**: Timeout configurável por webhook (padrão 30s)
7. **Estatísticas**: Tracking automático de performance e taxas de sucesso
