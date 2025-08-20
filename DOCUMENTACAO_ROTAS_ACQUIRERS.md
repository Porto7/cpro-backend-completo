# Documentação - Sistema de Adquirentes (routes/acquirers.js)

## Visão Geral
Sistema completo para gerenciamento de adquirentes de pagamento que permite configurar múltiplas empresas processadoras (PIX, cartão, etc.), alternar entre elas dinamicamente e monitorar a saúde dos serviços.

## Características Principais
- **Múltiplas Adquirentes**: Suporte a várias empresas processadoras
- **Failover Automático**: Alternância automática em caso de falha
- **Configuração Flexível**: APIs, webhooks, taxas e limites configuráveis
- **Health Check**: Monitoramento da saúde das APIs
- **Priorização**: Sistema de prioridade para escolha automática
- **Administração Completa**: CRUD completo para administradores

## Estrutura de Dados

### Modelo PaymentAcquirer
```javascript
{
  id: "uuid",
  name: "string", // Nome da adquirente
  slug: "string", // Identificador único
  description: "string", // Descrição
  logo_url: "string", // URL do logo
  supported_methods: ["pix", "credit_card"], // Métodos suportados
  api_config: {
    base_url: "string",
    api_key: "string",
    secret_key: "string",
    timeout: "number"
  },
  webhook_config: {
    url: "string",
    secret: "string",
    events: ["array"]
  },
  fee_config: {
    pix: { percentage: 1.5, fixed: 0 },
    credit_card: { percentage: 3.5, fixed: 0.39 }
  },
  limits_config: {
    min_amount: 100, // centavos
    max_amount: 5000000, // centavos
    daily_limit: 10000000
  },
  metadata: "object", // Dados adicionais
  priority: "number", // Prioridade (maior = preferencial)
  is_active: "boolean",
  is_default: "boolean",
  status: "active|inactive|maintenance",
  health_status: "healthy|warning|error",
  last_health_check: "datetime",
  created_at: "datetime",
  updated_at: "datetime"
}
```

## Rotas Implementadas

### 1. Rotas Públicas

#### `GET /api/acquirers/active`
**Descrição**: Lista adquirentes ativas para seleção no checkout
**Acesso**: Público

**Response Success**:
```json
{
  "status": "success",
  "count": 3,
  "acquirers": [
    {
      "id": "acq-123",
      "name": "PaymentCorp",
      "slug": "paymentcorp",
      "description": "Processadora principal PIX",
      "logo_url": "https://exemplo.com/logo-paymentcorp.png",
      "supported_methods": ["pix", "credit_card"],
      "fee_config": {
        "pix": { "percentage": 1.5, "fixed": 0 },
        "credit_card": { "percentage": 3.5, "fixed": 39 }
      },
      "limits_config": {
        "min_amount": 100,
        "max_amount": 5000000,
        "daily_limit": 10000000
      },
      "priority": 100,
      "is_active": true,
      "is_default": true,
      "status": "active",
      "health_status": "healthy"
    },
    {
      "id": "acq-456",
      "name": "FastPay",
      "slug": "fastpay",
      "description": "Processadora de backup",
      "supported_methods": ["pix"],
      "priority": 50,
      "is_active": true,
      "is_default": false,
      "status": "active",
      "health_status": "healthy"
    }
  ],
  "timestamp": "2024-01-01T10:00:00Z"
}
```

**Notas**: 
- Configurações da API (`api_config`) nunca são expostas publicamente
- Apenas adquirentes com `is_active: true` e `status: 'active'` são retornadas
- Ordenação por prioridade (maior primeiro) e nome

#### `GET /api/acquirers/default`
**Descrição**: Obtém adquirente padrão do sistema
**Acesso**: Público

**Response Success**:
```json
{
  "status": "success",
  "acquirer": {
    "id": "acq-123",
    "name": "PaymentCorp",
    "slug": "paymentcorp",
    "description": "Processadora principal PIX",
    "supported_methods": ["pix", "credit_card"],
    "is_default": true,
    "priority": 100,
    "health_status": "healthy"
  },
  "timestamp": "2024-01-01T10:00:00Z"
}
```

### 2. Rotas Administrativas

#### `GET /api/acquirers`
**Descrição**: Lista todas as adquirentes (admin)
**Acesso**: Admin

**Response Success**:
```json
{
  "status": "success",
  "count": 5,
  "acquirers": [
    {
      "id": "acq-123",
      "name": "PaymentCorp",
      "slug": "paymentcorp",
      "description": "Processadora principal PIX",
      "logo_url": "https://exemplo.com/logo.png",
      "supported_methods": ["pix", "credit_card"],
      "api_config": {
        "base_url": "https://api.paymentcorp.com",
        "api_key": "pk_live_xxx",
        "secret_key": "sk_live_xxx",
        "timeout": 30000
      },
      "webhook_config": {
        "url": "https://meusite.com/webhooks/paymentcorp",
        "secret": "wh_secret_xxx",
        "events": ["payment.approved", "payment.refused"]
      },
      "fee_config": {
        "pix": { "percentage": 1.5, "fixed": 0 },
        "credit_card": { "percentage": 3.5, "fixed": 39 }
      },
      "limits_config": {
        "min_amount": 100,
        "max_amount": 5000000,
        "daily_limit": 10000000
      },
      "metadata": {
        "contract_number": "CT-2024-001",
        "support_email": "suporte@paymentcorp.com"
      },
      "priority": 100,
      "is_active": true,
      "is_default": true,
      "status": "active",
      "health_status": "healthy",
      "last_health_check": "2024-01-01T09:55:00Z",
      "created_at": "2024-01-01T08:00:00Z",
      "updated_at": "2024-01-01T09:55:00Z"
    }
  ],
  "timestamp": "2024-01-01T10:00:00Z"
}
```

#### `POST /api/acquirers`
**Descrição**: Cria nova adquirente
**Acesso**: Admin

```javascript
// Payload de exemplo
{
  "name": "NovaProcessadora",
  "slug": "nova-processadora",
  "description": "Processadora especializada em PIX",
  "logo_url": "https://exemplo.com/logo-nova.png",
  "supported_methods": ["pix"],
  "api_config": {
    "base_url": "https://api.novaprocessadora.com",
    "api_key": "pk_live_abc123",
    "secret_key": "sk_live_def456",
    "timeout": 30000,
    "additional_headers": {
      "X-Custom-Header": "valor"
    }
  },
  "webhook_config": {
    "url": "https://meusite.com/webhooks/nova",
    "secret": "webhook_secret_789",
    "events": ["payment.completed", "payment.failed"],
    "retry_attempts": 3
  },
  "fee_config": {
    "pix": {
      "percentage": 1.2,
      "fixed": 0,
      "min_fee": 10
    }
  },
  "limits_config": {
    "min_amount": 500,
    "max_amount": 10000000,
    "daily_limit": 50000000,
    "monthly_limit": 1000000000
  },
  "metadata": {
    "contract_number": "CT-2024-002",
    "support_email": "suporte@novaprocessadora.com",
    "documentation_url": "https://docs.novaprocessadora.com"
  },
  "priority": 80,
  "is_default": false
}
```

**Response Success**:
```json
{
  "status": "success",
  "message": "Adquirente criada com sucesso",
  "acquirer": {
    "id": "acq-new-789",
    "name": "NovaProcessadora",
    "slug": "nova-processadora",
    "is_active": true,
    "status": "active",
    "created_at": "2024-01-01T10:05:00Z"
  },
  "timestamp": "2024-01-01T10:05:00Z"
}
```

**Validações**:
- Nome, slug e api_config são obrigatórios
- Slug deve ser único
- Se `is_default: true`, remove padrão de outras adquirentes

#### `GET /api/acquirers/:id`
**Descrição**: Obtém adquirente específica
**Acesso**: Admin

#### `PUT /api/acquirers/:id`
**Descrição**: Atualiza adquirente
**Acesso**: Admin

```javascript
// Payload de exemplo (campos opcionais)
{
  "name": "PaymentCorp Pro",
  "description": "Processadora principal com novos recursos",
  "is_active": true,
  "priority": 110,
  "api_config": {
    "base_url": "https://api-v2.paymentcorp.com",
    "timeout": 45000
  },
  "fee_config": {
    "pix": { "percentage": 1.3, "fixed": 0 }
  }
}
```

#### `DELETE /api/acquirers/:id`
**Descrição**: Remove adquirente
**Acesso**: Admin

**Validações**:
- Não permite deletar se for a única adquirente ativa
- Se for a padrão, automaticamente define outra como padrão

### 3. Operações Especiais

#### `POST /api/acquirers/:id/set-default`
**Descrição**: Define adquirente como padrão
**Acesso**: Admin

**Response Success**:
```json
{
  "status": "success",
  "message": "Adquirente definida como padrão com sucesso",
  "acquirer": {
    "id": "acq-456",
    "name": "FastPay",
    "is_default": true
  },
  "timestamp": "2024-01-01T10:10:00Z"
}
```

**Validações**:
- Adquirente deve estar ativa (`is_active: true`)
- Status deve ser 'active'
- Remove automaticamente o padrão de outras

#### `POST /api/acquirers/:id/health-check`
**Descrição**: Verifica saúde da API da adquirente
**Acesso**: Admin

**Response Success**:
```json
{
  "status": "success",
  "message": "Verificação de saúde concluída",
  "health_status": "healthy", // healthy, warning, error
  "last_check": "2024-01-01T10:15:00Z",
  "timestamp": "2024-01-01T10:15:00Z"
}
```

**Health Status**:
- `healthy`: API funcionando normalmente
- `warning`: API com latência alta ou problemas menores
- `error`: API indisponível ou com erros críticos

### 4. Estatísticas

#### `GET /api/acquirers/stats/overview`
**Descrição**: Estatísticas gerais das adquirentes
**Acesso**: Admin

**Response Success**:
```json
{
  "status": "success",
  "stats": {
    "total_acquirers": 5,
    "active_acquirers": 4,
    "healthy_acquirers": 3,
    "health_percentage": 75,
    "acquirers": [
      {
        "id": "acq-123",
        "name": "PaymentCorp",
        "slug": "paymentcorp",
        "is_active": true,
        "health_status": "healthy",
        "total_transactions": 1250,
        "total_volume": 185000.50
      },
      {
        "id": "acq-456",
        "name": "FastPay",
        "slug": "fastpay",
        "is_active": true,
        "health_status": "warning",
        "total_transactions": 340,
        "total_volume": 45600.00
      }
    ]
  },
  "timestamp": "2024-01-01T10:20:00Z"
}
```

## Funcionalidades Avançadas

### 1. Sistema de Failover
```javascript
// Lógica de seleção automática de adquirente
function selectBestAcquirer(amount, method) {
  const candidates = acquirers.filter(acq => 
    acq.is_active && 
    acq.status === 'active' &&
    acq.health_status === 'healthy' &&
    acq.supported_methods.includes(method) &&
    amount >= acq.limits_config.min_amount &&
    amount <= acq.limits_config.max_amount
  );
  
  // Ordenar por prioridade
  return candidates.sort((a, b) => b.priority - a.priority)[0];
}
```

### 2. Configuração Dinâmica
```javascript
// Exemplo de configuração flexível
{
  "api_config": {
    "base_url": "https://api.processadora.com",
    "api_key": "{{API_KEY}}", // Suporte a variáveis
    "endpoints": {
      "create_payment": "/v1/payments",
      "get_payment": "/v1/payments/{id}",
      "cancel_payment": "/v1/payments/{id}/cancel"
    },
    "headers": {
      "Authorization": "Bearer {{API_KEY}}",
      "Content-Type": "application/json",
      "X-API-Version": "2024-01"
    },
    "timeout": 30000,
    "retry_config": {
      "max_attempts": 3,
      "backoff_factor": 2
    }
  }
}
```

### 3. Monitoramento de Saúde
```javascript
// Health check automático
class AcquirerHealthMonitor {
  async checkHealth(acquirer) {
    try {
      const start = Date.now();
      const response = await fetch(`${acquirer.api_config.base_url}/health`, {
        timeout: acquirer.api_config.timeout || 30000
      });
      
      const latency = Date.now() - start;
      
      if (!response.ok) {
        return { status: 'error', latency, error: response.statusText };
      }
      
      if (latency > 5000) {
        return { status: 'warning', latency, message: 'High latency' };
      }
      
      return { status: 'healthy', latency };
    } catch (error) {
      return { status: 'error', error: error.message };
    }
  }
}
```

## Integração com Sistema de Pagamento

### 1. Seleção Automática
```javascript
// Service de pagamento
class PaymentService {
  async createPayment(orderData) {
    // Selecionar melhor adquirente
    const acquirer = await this.selectAcquirer(
      orderData.amount,
      orderData.method
    );
    
    if (!acquirer) {
      throw new Error('Nenhuma adquirente disponível');
    }
    
    // Processar pagamento
    return this.processPayment(orderData, acquirer);
  }
  
  async selectAcquirer(amount, method) {
    const acquirers = await PaymentAcquirer.findAll({
      where: {
        is_active: true,
        status: 'active',
        health_status: 'healthy',
        supported_methods: { [Op.contains]: [method] }
      },
      order: [['priority', 'DESC']]
    });
    
    return acquirers.find(acq => 
      amount >= acq.limits_config.min_amount &&
      amount <= acq.limits_config.max_amount
    );
  }
}
```

### 2. Configuração de Webhook
```javascript
// Configuração dinâmica de webhooks
{
  "webhook_config": {
    "url": "https://meusite.com/webhooks/{acquirer_slug}",
    "secret": "webhook_secret_123",
    "events": [
      "payment.created",
      "payment.approved", 
      "payment.refused",
      "payment.cancelled"
    ],
    "retry_config": {
      "max_attempts": 5,
      "retry_delays": [1000, 2000, 4000, 8000, 16000]
    },
    "signature_type": "hmac_sha256",
    "custom_headers": {
      "X-Webhook-Source": "payment-system"
    }
  }
}
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Dados inválidos ou slug já existe |
| 401 | Token de autenticação inválido |
| 403 | Acesso negado (apenas administradores) |
| 404 | Adquirente não encontrada |
| 500 | Erro interno do servidor |

## Considerações de Segurança

1. **Credenciais API**: Nunca expostas em rotas públicas
2. **Webhook Secrets**: Protegidos e únicos por adquirente
3. **Health Checks**: Não expõem dados sensíveis
4. **Admin Only**: Operações críticas apenas para administradores
5. **Audit Log**: Todas as alterações são logadas

## Monitoramento e Alertas

### Métricas Importantes
- Taxa de sucesso por adquirente
- Latência média das APIs
- Volume de transações
- Distribuição de uso
- Frequência de failovers

### Alertas Recomendados
- Adquirente ficou indisponível
- Latência acima de 5 segundos
- Taxa de erro acima de 5%
- Limite diário próximo do máximo
- Falha em health check

## Frontend de Administração

### React Component Example
```jsx
import React, { useState, useEffect } from 'react';

const AcquirerManager = ({ token }) => {
  const [acquirers, setAcquirers] = useState([]);
  const [stats, setStats] = useState(null);
  
  useEffect(() => {
    fetchAcquirers();
    fetchStats();
  }, []);
  
  const fetchAcquirers = async () => {
    const response = await fetch('/api/acquirers', {
      headers: { 'Authorization': `Bearer ${token}` }
    });
    const data = await response.json();
    setAcquirers(data.acquirers);
  };
  
  const setAsDefault = async (acquirerId) => {
    await fetch(`/api/acquirers/${acquirerId}/set-default`, {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${token}` }
    });
    fetchAcquirers();
  };
  
  const checkHealth = async (acquirerId) => {
    const response = await fetch(`/api/acquirers/${acquirerId}/health-check`, {
      method: 'POST',
      headers: { 'Authorization': `Bearer ${token}` }
    });
    const data = await response.json();
    alert(`Status: ${data.health_status}`);
    fetchAcquirers();
  };
  
  return (
    <div className="acquirer-manager">
      <h2>Gerenciar Adquirentes</h2>
      
      <div className="stats-overview">
        <div className="stat">
          <h3>{stats?.total_acquirers}</h3>
          <p>Total</p>
        </div>
        <div className="stat">
          <h3>{stats?.active_acquirers}</h3>
          <p>Ativas</p>
        </div>
        <div className="stat">
          <h3>{stats?.health_percentage}%</h3>
          <p>Saudáveis</p>
        </div>
      </div>
      
      <div className="acquirers-list">
        {acquirers.map(acquirer => (
          <div key={acquirer.id} className="acquirer-card">
            <div className="header">
              <img src={acquirer.logo_url} alt={acquirer.name} />
              <h3>{acquirer.name}</h3>
              {acquirer.is_default && <span className="badge">Padrão</span>}
            </div>
            
            <div className="status">
              <span className={`health ${acquirer.health_status}`}>
                {acquirer.health_status}
              </span>
              <span className={`active ${acquirer.is_active}`}>
                {acquirer.is_active ? 'Ativa' : 'Inativa'}
              </span>
            </div>
            
            <div className="methods">
              {acquirer.supported_methods.map(method => (
                <span key={method} className="method">{method}</span>
              ))}
            </div>
            
            <div className="actions">
              <button onClick={() => setAsDefault(acquirer.id)}>
                Definir como Padrão
              </button>
              <button onClick={() => checkHealth(acquirer.id)}>
                Verificar Saúde
              </button>
            </div>
          </div>
        ))}
      </div>
    </div>
  );
};
```

Este sistema de adquirentes oferece uma solução robusta e flexível para gerenciar múltiplas empresas processadoras de pagamento, com failover automático e monitoramento contínuo da saúde dos serviços.
