# Documentação - Sistema PIX Unificado (routes/pix.js)

## Visão Geral
Sistema unificado para processamento de pagamentos PIX com seleção automática ou manual de adquirentes, verificação de status e webhooks centralizados.

## Características Principais
- **PIX Simples**: Criação de pagamentos PIX sem split
- **Seleção de Adquirentes**: Manual ou automática baseada em prioridade
- **Status Unificado**: Verificação centralizada de status de pagamentos
- **Webhooks Centralizados**: Endpoint único para todos os adquirentes
- **Health Check**: Monitoramento da saúde do sistema
- **Failover Automático**: Seleção automática de adquirente saudável

## Status de Pagamentos

| Status | Descrição |
|--------|-----------|
| `pending` | Pagamento criado, aguardando |
| `processing` | Processando pagamento |
| `paid` | Pagamento confirmado |
| `cancelled` | Pagamento cancelado |
| `failed` | Pagamento falhou |
| `expired` | Pagamento expirou |

## Health Status de Adquirentes

| Status | Descrição |
|--------|-----------|
| `healthy` | Adquirente operando normalmente |
| `degraded` | Adquirente com problemas intermitentes |
| `unhealthy` | Adquirente com problemas graves |
| `offline` | Adquirente indisponível |

## Rotas Implementadas

### 1. Criação de PIX

#### `POST /api/pix/create`
**Descrição**: Criar pagamento PIX simples
**Acesso**: Público (autenticação opcional)

```javascript
// Payload de exemplo - Seleção automática de adquirente
{
  "productId": "product-123",
  "customer": {
    "name": "João Silva",
    "email": "joao@email.com",
    "phone": "+5511999999999",
    "document": "12345678901"
  }
}

// Payload de exemplo - Seleção manual de adquirente
{
  "productId": "product-123",
  "orderId": "order-456", // opcional, usar pedido existente
  "customer": {
    "name": "João Silva",
    "email": "joao@email.com",
    "phone": "+5511999999999",
    "document": "12345678901"
  },
  "acquirerSlug": "mercadopago" // seleção manual
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "PIX criado com sucesso",
  "data": {
    "orderId": "order-789",
    "paymentId": "mp_payment_123456",
    "qrCode": "00020126360014BR.GOV.BCB.PIX...",
    "pixKey": "12345678-1234-1234-1234-123456789012",
    "pixCopyPaste": "00020126360014BR.GOV.BCB.PIX...",
    "amount": 197.00,
    "currency": "BRL",
    "description": "Pagamento - Curso de JavaScript",
    "expiresAt": "2024-01-01T12:30:00Z",
    "acquirer": {
      "name": "Mercado Pago",
      "slug": "mercadopago"
    },
    "status": "pending",
    "createdAt": "2024-01-01T12:00:00Z",
    "checkStatusUrl": "/api/pix/status/order-789",
    "webhookUrl": "/api/pix/webhook/mercadopago"
  }
}
```

**Response Error - Produto não encontrado**:
```json
{
  "error": "Produto não encontrado"
}
```

**Response Error - Adquirente indisponível**:
```json
{
  "error": "Nenhuma adquirente ativa encontrada"
}
```

**Response Error - Validação**:
```json
{
  "error": "Customer deve ter: name, email"
}
```

### 2. Verificação de Status

#### `GET /api/pix/status/:orderId`
**Descrição**: Verificar status atual do pagamento PIX
**Acesso**: Público

**Response Success - Pendente**:
```json
{
  "success": true,
  "data": {
    "orderId": "order-789",
    "paymentId": "mp_payment_123456",
    "status": "pending",
    "paid": false,
    "amount": 197.00,
    "currency": "BRL",
    "paymentMethod": "pix",
    "qrCode": "00020126360014BR.GOV.BCB.PIX...",
    "pixKey": "12345678-1234-1234-1234-123456789012",
    "expiresAt": "2024-01-01T12:30:00Z",
    "acquirer": {
      "name": "Mercado Pago",
      "slug": "mercadopago"
    },
    "createdAt": "2024-01-01T12:00:00Z",
    "updatedAt": "2024-01-01T12:00:00Z",
    "paidAt": null,
    "customer": {
      "name": "João Silva",
      "email": "joao@email.com",
      "phone": "+5511999999999"
    },
    "product": {
      "id": "product-123",
      "name": "Curso de JavaScript",
      "price": 197.00
    }
  }
}
```

**Response Success - Pago**:
```json
{
  "success": true,
  "data": {
    "orderId": "order-789",
    "paymentId": "mp_payment_123456",
    "status": "paid",
    "paid": true,
    "amount": 197.00,
    "currency": "BRL",
    "paymentMethod": "pix",
    "qrCode": "00020126360014BR.GOV.BCB.PIX...",
    "pixKey": "12345678-1234-1234-1234-123456789012",
    "expiresAt": "2024-01-01T12:30:00Z",
    "acquirer": {
      "name": "Mercado Pago",
      "slug": "mercadopago"
    },
    "createdAt": "2024-01-01T12:00:00Z",
    "updatedAt": "2024-01-01T12:15:00Z",
    "paidAt": "2024-01-01T12:15:00Z",
    "customer": {
      "name": "João Silva",
      "email": "joao@email.com",
      "phone": "+5511999999999"
    },
    "product": {
      "id": "product-123",
      "name": "Curso de JavaScript",
      "price": 197.00
    }
  }
}
```

**Response Error**:
```json
{
  "error": "Pedido não encontrado"
}
```

### 3. Webhooks Centralizados

#### `POST /api/pix/webhook/:acquirerSlug`
**Descrição**: Receber notificações de pagamento de qualquer adquirente
**Acesso**: Sistema (chamado pelas adquirentes)

**Headers Esperados**:
```
Content-Type: application/json
X-Webhook-Signature: sha256=abc123... (opcional, se configurado)
X-Signature: sha256=abc123... (alternativo)
```

**Payload de Exemplo - Mercado Pago**:
```json
{
  "id": "mp_payment_123456",
  "topic": "payment",
  "type": "payment",
  "action": "payment.updated",
  "data": {
    "id": "mp_payment_123456"
  }
}
```

**Payload de Exemplo - PagSeguro**:
```json
{
  "reference": "order-789",
  "code": "ps_transaction_654321",
  "status": "3"
}
```

**Payload de Exemplo - Asaas**:
```json
{
  "id": "asaas_payment_987654",
  "externalReference": "order-789",
  "status": "CONFIRMED",
  "value": 197.00
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Webhook processado com sucesso",
  "orderId": "order-789"
}
```

**Response Error - Assinatura inválida**:
```json
{
  "success": false,
  "error": "Assinatura inválida"
}
```

### 4. Endpoints Auxiliares

#### `GET /api/pix/acquirers`
**Descrição**: Listar adquirentes disponíveis
**Acesso**: Público

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "slug": "mercadopago",
      "name": "Mercado Pago",
      "isDefault": true,
      "healthStatus": "healthy",
      "supportsPix": true
    },
    {
      "slug": "pagseguro",
      "name": "PagSeguro",
      "isDefault": false,
      "healthStatus": "healthy",
      "supportsPix": true
    },
    {
      "slug": "asaas",
      "name": "Asaas",
      "isDefault": false,
      "healthStatus": "degraded",
      "supportsPix": true
    }
  ]
}
```

#### `GET /api/pix/health`
**Descrição**: Health check do sistema PIX
**Acesso**: Público

**Response Success**:
```json
{
  "success": true,
  "status": "healthy",
  "data": {
    "acquirers": {
      "total": 3,
      "healthy": 2,
      "healthPercentage": 67
    },
    "orders": {
      "last24h": 145
    },
    "timestamp": "2024-01-01T12:00:00Z"
  }
}
```

**Response Error**:
```json
{
  "success": false,
  "status": "unhealthy",
  "error": "Erro no health check"
}
```

## Integração Frontend

### Componente de Checkout PIX
```jsx
import React, { useState, useEffect } from 'react';

const PixCheckout = ({ productId, customer, onSuccess, onError }) => {
  const [pixData, setPixData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [status, setStatus] = useState('idle');
  const [acquirers, setAcquirers] = useState([]);
  const [selectedAcquirer, setSelectedAcquirer] = useState('');
  
  useEffect(() => {
    loadAcquirers();
  }, []);
  
  const loadAcquirers = async () => {
    try {
      const response = await fetch('/api/pix/acquirers');
      const data = await response.json();
      setAcquirers(data.data);
      // Selecionar automaticamente o padrão
      const defaultAcquirer = data.data.find(a => a.isDefault);
      if (defaultAcquirer) {
        setSelectedAcquirer(defaultAcquirer.slug);
      }
    } catch (error) {
      console.error('Erro ao carregar adquirentes:', error);
    }
  };
  
  const createPix = async () => {
    setLoading(true);
    setStatus('creating');
    
    try {
      const payload = {
        productId,
        customer
      };
      
      // Adicionar adquirente se selecionado manualmente
      if (selectedAcquirer) {
        payload.acquirerSlug = selectedAcquirer;
      }
      
      const response = await fetch('/api/pix/create', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });
      
      const data = await response.json();
      
      if (data.success) {
        setPixData(data.data);
        setStatus('pending');
        startStatusPolling(data.data.orderId);
      } else {
        setStatus('error');
        onError?.(data.error);
      }
    } catch (error) {
      setStatus('error');
      onError?.(error.message);
    } finally {
      setLoading(false);
    }
  };
  
  const startStatusPolling = (orderId) => {
    const checkStatus = async () => {
      try {
        const response = await fetch(`/api/pix/status/${orderId}`);
        const data = await response.json();
        
        if (data.success) {
          if (data.data.paid) {
            setStatus('paid');
            onSuccess?.(data.data);
            return; // Parar polling
          }
          
          // Continuar polling se ainda pendente
          setTimeout(checkStatus, 3000); // Verificar a cada 3 segundos
        }
      } catch (error) {
        console.error('Erro ao verificar status:', error);
        // Continuar tentando mesmo com erro
        setTimeout(checkStatus, 5000);
      }
    };
    
    // Iniciar polling após 3 segundos
    setTimeout(checkStatus, 3000);
  };
  
  const copyPixCode = () => {
    navigator.clipboard.writeText(pixData.qrCode);
    alert('Código PIX copiado para a área de transferência!');
  };
  
  const downloadQRCode = () => {
    // Gerar QR Code como imagem e fazer download
    const canvas = document.createElement('canvas');
    const qrCode = new QRCodeGenerator(canvas, {
      text: pixData.qrCode,
      width: 256,
      height: 256
    });
    
    const link = document.createElement('a');
    link.download = `pix-qrcode-${pixData.orderId}.png`;
    link.href = canvas.toDataURL();
    link.click();
  };
  
  return (
    <div className="pix-checkout">
      <h2>💰 Pagamento PIX</h2>
      
      {status === 'idle' && (
        <div className="checkout-setup">
          <div className="acquirer-selection">
            <h3>🏦 Escolher Adquirente</h3>
            <select 
              value={selectedAcquirer}
              onChange={(e) => setSelectedAcquirer(e.target.value)}
            >
              <option value="">Seleção automática (recomendado)</option>
              {acquirers
                .filter(a => a.healthStatus === 'healthy')
                .map(acquirer => (
                <option key={acquirer.slug} value={acquirer.slug}>
                  {acquirer.name} {acquirer.isDefault ? '(Padrão)' : ''}
                </option>
              ))}
            </select>
            <small>
              {selectedAcquirer 
                ? `Usar ${acquirers.find(a => a.slug === selectedAcquirer)?.name}`
                : 'Sistema escolherá automaticamente a melhor opção'
              }
            </small>
          </div>
          
          <button 
            onClick={createPix}
            disabled={loading}
            className="btn btn-primary btn-large"
          >
            {loading ? 'Gerando PIX...' : '🎯 Gerar PIX'}
          </button>
        </div>
      )}
      
      {status === 'creating' && (
        <div className="creating-status">
          <div className="spinner"></div>
          <p>Gerando seu PIX...</p>
        </div>
      )}
      
      {status === 'pending' && pixData && (
        <div className="pix-payment">
          <div className="pix-info">
            <h3>✅ PIX Gerado!</h3>
            <p>Valor: <strong>R$ {pixData.amount.toFixed(2)}</strong></p>
            <p>Expira em: <strong>{new Date(pixData.expiresAt).toLocaleString()}</strong></p>
            <p>Adquirente: <strong>{pixData.acquirer.name}</strong></p>
          </div>
          
          <div className="qr-code-section">
            <div className="qr-code">
              <QRCodeDisplay text={pixData.qrCode} size={200} />
            </div>
            
            <div className="qr-actions">
              <button onClick={copyPixCode} className="btn btn-secondary">
                📋 Copiar Código
              </button>
              <button onClick={downloadQRCode} className="btn btn-secondary">
                💾 Baixar QR Code
              </button>
            </div>
          </div>
          
          <div className="copy-paste-section">
            <h4>💬 Ou use o PIX Copia e Cola:</h4>
            <div className="copy-paste-box">
              <textarea
                readOnly
                value={pixData.pixCopyPaste}
                onClick={(e) => e.target.select()}
              />
            </div>
          </div>
          
          <div className="payment-status">
            <div className="status-indicator">
              <div className="pulse"></div>
              <span>Aguardando pagamento...</span>
            </div>
            <p>Atualizamos automaticamente quando o pagamento for confirmado.</p>
          </div>
        </div>
      )}
      
      {status === 'paid' && (
        <div className="payment-success">
          <div className="success-icon">✅</div>
          <h3>Pagamento Confirmado!</h3>
          <p>Seu pagamento PIX foi processado com sucesso.</p>
        </div>
      )}
      
      {status === 'error' && (
        <div className="payment-error">
          <div className="error-icon">❌</div>
          <h3>Erro no Pagamento</h3>
          <p>Houve um problema ao gerar o PIX. Tente novamente.</p>
          <button onClick={() => setStatus('idle')} className="btn btn-primary">
            🔄 Tentar Novamente
          </button>
        </div>
      )}
    </div>
  );
};
```

### Hook para PIX
```jsx
// Hook personalizado para PIX
const usePix = () => {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const createPix = async (productId, customer, acquirerSlug = null) => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch('/api/pix/create', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          productId,
          customer,
          acquirerSlug
        })
      });
      
      const data = await response.json();
      
      if (!data.success) {
        throw new Error(data.error);
      }
      
      return data.data;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  const checkStatus = async (orderId) => {
    try {
      const response = await fetch(`/api/pix/status/${orderId}`);
      const data = await response.json();
      
      if (!data.success) {
        throw new Error(data.error);
      }
      
      return data.data;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  };
  
  const getAcquirers = async () => {
    try {
      const response = await fetch('/api/pix/acquirers');
      const data = await response.json();
      return data.data;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  };
  
  return {
    createPix,
    checkStatus,
    getAcquirers,
    loading,
    error
  };
};
```

### Componente de Status PIX
```jsx
const PixStatus = ({ orderId }) => {
  const [status, setStatus] = useState(null);
  const [loading, setLoading] = useState(true);
  const { checkStatus } = usePix();
  
  useEffect(() => {
    loadStatus();
    
    // Polling automático para status pendente
    const interval = setInterval(() => {
      if (status && !status.paid) {
        loadStatus();
      }
    }, 5000);
    
    return () => clearInterval(interval);
  }, [orderId, status]);
  
  const loadStatus = async () => {
    try {
      const statusData = await checkStatus(orderId);
      setStatus(statusData);
    } catch (error) {
      console.error('Erro ao carregar status:', error);
    } finally {
      setLoading(false);
    }
  };
  
  if (loading) {
    return <div>Carregando status...</div>;
  }
  
  if (!status) {
    return <div>Erro ao carregar status do pagamento.</div>;
  }
  
  return (
    <div className={`pix-status ${status.status}`}>
      <div className="status-header">
        <h3>Status do Pagamento PIX</h3>
        <span className={`status-badge ${status.status}`}>
          {getStatusLabel(status.status)}
        </span>
      </div>
      
      <div className="payment-info">
        <p><strong>ID do Pedido:</strong> {status.orderId}</p>
        <p><strong>Valor:</strong> R$ {status.amount.toFixed(2)}</p>
        <p><strong>Método:</strong> PIX</p>
        <p><strong>Adquirente:</strong> {status.acquirer.name}</p>
      </div>
      
      {!status.paid && status.qrCode && (
        <div className="payment-pending">
          <p>⏳ Aguardando pagamento...</p>
          <p>Expira em: {new Date(status.expiresAt).toLocaleString()}</p>
        </div>
      )}
      
      {status.paid && (
        <div className="payment-success">
          <p>✅ Pagamento confirmado!</p>
          <p>Pago em: {new Date(status.paidAt).toLocaleString()}</p>
        </div>
      )}
    </div>
  );
  
  function getStatusLabel(status) {
    const labels = {
      pending: 'Pendente',
      processing: 'Processando',
      paid: 'Pago',
      cancelled: 'Cancelado',
      failed: 'Falhou',
      expired: 'Expirado'
    };
    return labels[status] || status;
  }
};
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Parâmetros obrigatórios ausentes ou inválidos |
| 401 | Assinatura do webhook inválida |
| 404 | Produto, pedido ou adquirente não encontrado |
| 500 | Erro interno do servidor ou falha na adquirente |

## Considerações de Segurança

1. **Validação de Webhooks**: Verificação de assinatura quando configurada
2. **Timeout de PIX**: Expiração automática após 30 minutos
3. **Validação de Dados**: Verificação rigorosa dos dados do cliente
4. **Health Check**: Monitoramento contínuo das adquirentes
5. **Logs Detalhados**: Auditoria completa de todas as transações

## Melhores Práticas

1. **Polling Inteligente**: Verificar status a cada 3-5 segundos
2. **Fallback de Adquirentes**: Ter múltiplas opções configuradas
3. **UX Responsiva**: Feedback visual claro para o usuário
4. **Timeout Handling**: Lidar graciosamente com PIX expirados
5. **Monitoring**: Acompanhar taxa de sucesso por adquirente

Este sistema oferece uma solução robusta e unificada para pagamentos PIX, com flexibilidade para diferentes adquirentes e experiência otimizada para o usuário.
