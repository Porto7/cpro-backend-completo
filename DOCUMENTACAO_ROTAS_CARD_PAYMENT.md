# Documentação - Sistema de Pagamento com Cartão (routes/cardPayment.js)

## Visão Geral
Sistema completo de pagamento com cartão de crédito/débito que suporta split payment automático, gestão de comissões e webhooks para atualizações de status em tempo real.

## Características Principais
- **Split Payment**: Divisão automática de pagamentos
- **Webhooks**: Atualizações em tempo real
- **Comissões**: Sistema automático de comissões para parceiros
- **Segurança**: Validação de assinatura de webhooks
- **Gestão Completa**: Criação, captura, cancelamento e estorno

## Configuração

### Variáveis de Ambiente
```bash
# Autenticação
JWT_SECRET=sua_chave_jwt_secreta

# Webhook Security
CARD_WEBHOOK_SECRET=sua_chave_webhook_secreta

# Ambiente
NODE_ENV=production|development
```

### Middleware de Segurança
- **Autenticação Opcional**: JWT token para algumas operações
- **Validação de Webhook**: Verificação de assinatura HMAC SHA256
- **CORS**: Configuração adequada para chamadas frontend

## Rotas Implementadas

### 1. Criação de Pagamento

#### `POST /api/card-payment/create`
**Descrição**: Cria pagamento com cartão e split automático
**Acesso**: Público (com autenticação opcional)

```javascript
// Payload de exemplo
{
  "orderId": "order-123",
  "partnerCode": "PARTNER123", // Opcional
  "cardData": {
    "cardNumber": "4111111111111111",
    "expMonth": "12",
    "expYear": "2025",
    "cvc": "123",
    "holderName": "João Silva"
  },
  "customerData": { // Opcional - atualiza dados do cliente
    "phone": "(11) 99999-9999",
    "address": {
      "street": "Rua das Flores, 123",
      "city": "São Paulo",
      "state": "SP",
      "zip_code": "01234-567"
    }
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Pagamento criado com sucesso",
  "data": {
    "charge_id": "charge_abc123",
    "status": "pending",
    "amount": 19700,
    "currency": "BRL",
    "splits": [
      {
        "recipient_id": "seller_123",
        "amount": 17730,
        "percentage": 90
      },
      {
        "recipient_id": "platform",
        "amount": 1970,
        "percentage": 10
      }
    ],
    "metadata": {
      "order_id": "order-123",
      "partner_code": "PARTNER123"
    },
    "created_at": "2024-01-01T10:00:00Z"
  }
}
```

**Validações**:
- `orderId` obrigatório
- `cardData` completo obrigatório
- Pedido deve estar com status `pending`
- Código de parceiro validado se fornecido

### 2. Consulta de Status

#### `GET /api/card-payment/status/:chargeId`
**Descrição**: Consulta status atual do pagamento
**Acesso**: Público

**Response Success**:
```json
{
  "success": true,
  "data": {
    "charge_id": "charge_abc123",
    "status": "succeeded", // pending, succeeded, failed, captured, voided, refunded
    "amount": 19700,
    "amount_captured": 19700,
    "amount_refunded": 0,
    "payment_method": {
      "type": "credit_card",
      "brand": "visa",
      "last_four": "1111",
      "exp_month": 12,
      "exp_year": 2025
    },
    "customer": {
      "name": "João Silva",
      "email": "joao@exemplo.com"
    },
    "splits": [...],
    "created_at": "2024-01-01T10:00:00Z",
    "paid_at": "2024-01-01T10:05:00Z"
  }
}
```

**Status Possíveis**:
- `pending`: Aguardando processamento
- `succeeded`: Autorizado com sucesso
- `failed`: Recusado/Falhou
- `captured`: Capturado
- `voided`: Cancelado
- `refunded`: Estornado

### 3. Captura de Pagamento

#### `POST /api/card-payment/capture/:chargeId`
**Descrição**: Captura pagamento pré-autorizado
**Acesso**: Autenticado (opcional)

```javascript
// Payload de exemplo
{
  "amount": 19700 // Opcional - valor em centavos a capturar
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Pagamento capturado com sucesso",
  "data": {
    "charge_id": "charge_abc123",
    "status": "captured",
    "amount_captured": 19700,
    "captured_at": "2024-01-01T10:10:00Z"
  }
}
```

### 4. Cancelamento de Pagamento

#### `POST /api/card-payment/void/:chargeId`
**Descrição**: Cancela pagamento autorizado (antes da captura)
**Acesso**: Autenticado (opcional)

**Response Success**:
```json
{
  "success": true,
  "message": "Pagamento cancelado com sucesso",
  "data": {
    "charge_id": "charge_abc123",
    "status": "voided",
    "voided_at": "2024-01-01T10:15:00Z"
  }
}
```

### 5. Webhook de Atualizações

#### `POST /api/card-payment/webhook`
**Descrição**: Recebe atualizações da plataforma de pagamento
**Acesso**: Webhook (validação de assinatura)

**Headers Obrigatórios**:
```
Content-Type: application/json
X-Webhook-Signature: sha256=assinatura_hmac
```

**Eventos Suportados**:

##### `charge.succeeded`
```json
{
  "type": "charge.succeeded",
  "data": {
    "id": "charge_abc123",
    "status": "succeeded",
    "amount": 19700,
    "metadata": {
      "order_id": "order-123",
      "partner_code": "PARTNER123"
    }
  }
}
```

##### `charge.failed`
```json
{
  "type": "charge.failed",
  "data": {
    "id": "charge_abc123",
    "status": "failed",
    "failure_reason": "insufficient_funds",
    "metadata": {
      "order_id": "order-123"
    }
  }
}
```

##### `charge.captured`
```json
{
  "type": "charge.captured",
  "data": {
    "id": "charge_abc123",
    "status": "captured",
    "amount_captured": 19700,
    "metadata": {
      "order_id": "order-123"
    }
  }
}
```

##### `charge.voided`
```json
{
  "type": "charge.voided",
  "data": {
    "id": "charge_abc123",
    "status": "voided",
    "metadata": {
      "order_id": "order-123"
    }
  }
}
```

##### `charge.refunded`
```json
{
  "type": "charge.refunded",
  "data": {
    "id": "charge_abc123",
    "status": "refunded",
    "amount_refunded": 19700,
    "metadata": {
      "order_id": "order-123"
    }
  }
}
```

### 6. Utilitários

#### `GET /api/card-payment/test`
**Descrição**: Endpoint de teste para verificar conectividade
**Acesso**: Público

**Response Success**:
```json
{
  "success": true,
  "message": "API de pagamento com cartão funcionando",
  "timestamp": "2024-01-01T10:00:00Z",
  "webhook_url": "https://api.exemplo.com/api/card-payment/webhook"
}
```

#### `POST /api/card-payment/simulate-webhook` (Desenvolvimento)
**Descrição**: Simula webhook para testes
**Acesso**: Apenas em desenvolvimento

```javascript
// Payload de exemplo
{
  "type": "charge.succeeded",
  "chargeId": "charge_test_123",
  "orderId": "order_test_123",
  "status": "succeeded"
}
```

## Sistema de Split Payment

### Configuração Automática
O sistema automaticamente calcula e aplica splits baseado em:

1. **Taxa da Plataforma**: Percentual fixo para a plataforma
2. **Comissões de Parceiros**: Baseado no código do parceiro
3. **Valor do Vendedor**: Restante após deduções

### Exemplo de Split
```javascript
// Pedido de R$ 197,00 com parceiro 10%
{
  "total_amount": 19700, // R$ 197,00
  "splits": [
    {
      "recipient_id": "seller_123",
      "amount": 15760, // R$ 157,60 (80%)
      "description": "Valor do vendedor"
    },
    {
      "recipient_id": "partner_456",
      "amount": 1970, // R$ 19,70 (10%)
      "description": "Comissão do parceiro"
    },
    {
      "recipient_id": "platform",
      "amount": 1970, // R$ 19,70 (10%)
      "description": "Taxa da plataforma"
    }
  ]
}
```

## Segurança do Webhook

### Validação de Assinatura
```javascript
// Validação HMAC SHA256
const crypto = require('crypto');

function validateSignature(payload, signature, secret) {
  const expectedSignature = 'sha256=' + crypto
    .createHmac('sha256', secret)
    .update(payload, 'utf8')
    .digest('hex');
    
  return signature === expectedSignature;
}
```

### Headers de Segurança
- `X-Webhook-Signature`: Assinatura HMAC do payload
- `User-Agent`: Identificador da plataforma
- `X-Webhook-Id`: ID único do evento

## Integração Frontend

### JavaScript Example
```javascript
class CardPaymentService {
  constructor(apiUrl, token = null) {
    this.apiUrl = apiUrl;
    this.token = token;
  }
  
  async createPayment(orderId, cardData, partnerCode = null) {
    const response = await fetch(`${this.apiUrl}/api/card-payment/create`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        ...(this.token && { 'Authorization': `Bearer ${this.token}` })
      },
      body: JSON.stringify({
        orderId,
        cardData,
        partnerCode
      })
    });
    
    return response.json();
  }
  
  async getPaymentStatus(chargeId) {
    const response = await fetch(`${this.apiUrl}/api/card-payment/status/${chargeId}`);
    return response.json();
  }
  
  async capturePayment(chargeId, amount = null) {
    const response = await fetch(`${this.apiUrl}/api/card-payment/capture/${chargeId}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        ...(this.token && { 'Authorization': `Bearer ${this.token}` })
      },
      body: JSON.stringify({ amount })
    });
    
    return response.json();
  }
}
```

### React Component Example
```jsx
import React, { useState } from 'react';

const CardPaymentForm = ({ orderId, onSuccess, onError }) => {
  const [cardData, setCardData] = useState({
    cardNumber: '',
    expMonth: '',
    expYear: '',
    cvc: '',
    holderName: ''
  });
  const [loading, setLoading] = useState(false);
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    
    try {
      const paymentService = new CardPaymentService('/api');
      const result = await paymentService.createPayment(orderId, cardData);
      
      if (result.success) {
        onSuccess(result.data);
      } else {
        onError(result.error);
      }
    } catch (error) {
      onError(error.message);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        placeholder="Número do cartão"
        value={cardData.cardNumber}
        onChange={(e) => setCardData({...cardData, cardNumber: e.target.value})}
        maxLength="19"
      />
      
      <div className="exp-cvc">
        <input
          type="text"
          placeholder="MM"
          value={cardData.expMonth}
          onChange={(e) => setCardData({...cardData, expMonth: e.target.value})}
          maxLength="2"
        />
        <input
          type="text"
          placeholder="AAAA"
          value={cardData.expYear}
          onChange={(e) => setCardData({...cardData, expYear: e.target.value})}
          maxLength="4"
        />
        <input
          type="text"
          placeholder="CVC"
          value={cardData.cvc}
          onChange={(e) => setCardData({...cardData, cvc: e.target.value})}
          maxLength="4"
        />
      </div>
      
      <input
        type="text"
        placeholder="Nome no cartão"
        value={cardData.holderName}
        onChange={(e) => setCardData({...cardData, holderName: e.target.value})}
      />
      
      <button type="submit" disabled={loading}>
        {loading ? 'Processando...' : 'Finalizar Pagamento'}
      </button>
    </form>
  );
};
```

## Fluxo de Pagamento

### 1. Fluxo Normal
```
1. Cliente preenche dados do cartão
2. Frontend chama POST /create
3. Sistema processa e retorna charge_id
4. Webhook confirma status
5. Comissões são processadas
6. Cliente é redirecionado
```

### 2. Fluxo com Pré-autorização
```
1. Criar pagamento (status: pending)
2. Aguardar confirmação
3. Capturar quando necessário
4. Ou cancelar se não capturado
```

### 3. Fluxo de Estorno
```
1. Pagamento já capturado
2. Solicitação de estorno
3. Processamento automático
4. Webhook confirma estorno
5. Comissões são canceladas
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Dados inválidos ou pedido já processado |
| 401 | Assinatura de webhook inválida |
| 403 | Operação não permitida em produção |
| 404 | Pedido ou charge não encontrado |
| 500 | Erro interno do servidor ou processamento |

## Considerações de Segurança

1. **PCI Compliance**: Dados de cartão nunca armazenados
2. **Tokenização**: Uso de tokens para referências
3. **HTTPS**: Todas as comunicações criptografadas
4. **Webhook Security**: Validação rigorosa de assinaturas
5. **Rate Limiting**: Proteção contra ataques

## Monitoramento e Logs

### Eventos Logados
- Criação de pagamentos
- Aprovações e recusas
- Capturas e cancelamentos
- Webhooks recebidos
- Processamento de comissões

### Métricas Importantes
- Taxa de aprovação
- Tempo de processamento
- Volume de transações
- Chargeback rate
- Latência dos webhooks

## Troubleshooting

### Problemas Comuns
1. **Webhook não recebido**: Verificar URL e conectividade
2. **Assinatura inválida**: Verificar CARD_WEBHOOK_SECRET
3. **Pagamento recusado**: Verificar dados do cartão
4. **Split incorreto**: Verificar configuração de comissões

### Logs de Debug
```javascript
console.log('🔔 Webhook recebido:', event.type);
console.log('✅ Pagamento aprovado:', chargeData.id);
console.log('❌ Pagamento falhado:', chargeData.failure_reason);
```

Este sistema oferece uma solução completa e segura para processamento de pagamentos com cartão, incluindo funcionalidades avançadas como split payment automático e gestão de comissões.
