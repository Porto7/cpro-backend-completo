# Documentação - Sistema de Assinaturas (routes/subscriptions.js)

## Visão Geral
Motor de recorrência completo para gerenciamento de assinaturas e pagamentos recorrentes. Permite criar planos, gerenciar assinaturas e processar cobranças automáticas.

## Características Principais
- **Planos Flexíveis**: Múltiplos intervalos de cobrança
- **Gestão de Assinaturas**: Criação, cancelamento e reativação
- **Pagamentos Automáticos**: Processamento recorrente
- **Períodos de Teste**: Trials antes da cobrança
- **Analytics**: Estatísticas detalhadas de assinaturas
- **Processamento em Lote**: Cobrança automática via cron

## Intervalos de Cobrança

| Intervalo | Descrição | Frequência |
|-----------|-----------|------------|
| `daily` | Diário | A cada dia |
| `weekly` | Semanal | A cada semana |
| `monthly` | Mensal | A cada mês |
| `quarterly` | Trimestral | A cada 3 meses |
| `yearly` | Anual | A cada ano |

## Status de Assinaturas

| Status | Descrição | Cor |
|--------|-----------|-----|
| `trial` | Período de Teste | #17a2b8 (Azul) |
| `active` | Ativo | #28a745 (Verde) |
| `past_due` | Em Atraso | #ffc107 (Amarelo) |
| `cancelled` | Cancelado | #dc3545 (Vermelho) |
| `expired` | Expirado | #6c757d (Cinza) |

## Rotas Implementadas

### 1. Gestão de Planos

#### `POST /api/subscriptions/plans`
**Descrição**: Criar novo plano de assinatura
**Acesso**: Producer, Admin

```javascript
// Payload de exemplo
{
  "name": "Plano Premium Mensal",
  "description": "Acesso completo aos recursos premium",
  "product_id": "product-123",
  "interval": "monthly",
  "price": 49.90,
  "currency": "BRL",
  "trial_period_days": 7,
  "features": [
    "Produtos ilimitados",
    "Suporte prioritário",
    "Analytics avançadas",
    "API completa"
  ],
  "max_subscribers": null, // null = ilimitado
  "is_active": true
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Plano de assinatura criado com sucesso",
  "data": {
    "id": "plan-456",
    "name": "Plano Premium Mensal",
    "description": "Acesso completo aos recursos premium",
    "product_id": "product-123",
    "interval": "monthly",
    "price": 49.90,
    "currency": "BRL",
    "trial_period_days": 7,
    "features": [
      "Produtos ilimitados",
      "Suporte prioritário",
      "Analytics avançadas",
      "API completa"
    ],
    "max_subscribers": null,
    "current_subscribers": 0,
    "is_active": true,
    "created_at": "2024-01-01T12:00:00Z",
    "updated_at": "2024-01-01T12:00:00Z"
  }
}
```

**Response Error**:
```json
{
  "success": false,
  "message": "Nome, produto, intervalo e preço são obrigatórios"
}
```

#### `PUT /api/subscriptions/plans/:id`
**Descrição**: Atualizar plano de assinatura existente
**Acesso**: Producer, Admin

```javascript
// Payload de exemplo
{
  "name": "Plano Premium Mensal - Atualizado",
  "price": 59.90,
  "trial_period_days": 14,
  "features": [
    "Produtos ilimitados",
    "Suporte prioritário",
    "Analytics avançadas",
    "API completa",
    "Integrações personalizadas"
  ]
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Plano atualizado com sucesso",
  "data": {
    "id": "plan-456",
    "name": "Plano Premium Mensal - Atualizado",
    "price": 59.90,
    "trial_period_days": 14,
    "features": [
      "Produtos ilimitados",
      "Suporte prioritário",
      "Analytics avançadas",
      "API completa",
      "Integrações personalizadas"
    ],
    "updated_at": "2024-01-01T15:00:00Z"
  }
}
```

### 2. Gestão de Assinaturas

#### `POST /api/subscriptions`
**Descrição**: Criar nova assinatura para o usuário autenticado
**Acesso**: Autenticado

```javascript
// Payload de exemplo
{
  "planId": "plan-456",
  "paymentMethod": {
    "type": "credit_card",
    "card_token": "card_token_123",
    "save_card": true
  },
  "coupon_code": "DESCONTO10", // opcional
  "metadata": {
    "source": "website",
    "campaign": "promo_janeiro"
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Assinatura criada com sucesso",
  "data": {
    "id": "subscription-789",
    "user_id": "user-123",
    "plan_id": "plan-456",
    "status": "trial",
    "current_period_start": "2024-01-01T12:00:00Z",
    "current_period_end": "2024-01-08T12:00:00Z",
    "trial_end": "2024-01-08T12:00:00Z",
    "next_payment_date": "2024-01-08T12:00:00Z",
    "amount": 59.90,
    "currency": "BRL",
    "payment_method": {
      "type": "credit_card",
      "last_four": "1234",
      "brand": "visa"
    },
    "plan": {
      "id": "plan-456",
      "name": "Plano Premium Mensal - Atualizado",
      "interval": "monthly",
      "price": 59.90
    },
    "created_at": "2024-01-01T12:00:00Z"
  }
}
```

#### `GET /api/subscriptions/my`
**Descrição**: Listar assinaturas do usuário autenticado
**Acesso**: Autenticado

**Query Parameters**:
- `status`: Filtrar por status (trial, active, past_due, cancelled, expired)

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "id": "subscription-789",
      "status": "active",
      "current_period_start": "2024-01-08T12:00:00Z",
      "current_period_end": "2024-02-08T12:00:00Z",
      "next_payment_date": "2024-02-08T12:00:00Z",
      "amount": 59.90,
      "currency": "BRL",
      "plan": {
        "id": "plan-456",
        "name": "Plano Premium Mensal - Atualizado",
        "interval": "monthly"
      },
      "payment_method": {
        "type": "credit_card",
        "last_four": "1234",
        "brand": "visa"
      },
      "created_at": "2024-01-01T12:00:00Z",
      "payments_history": [
        {
          "id": "payment-101",
          "amount": 59.90,
          "status": "paid",
          "paid_at": "2024-01-08T12:00:05Z"
        }
      ]
    }
  ]
}
```

#### `PUT /api/subscriptions/:id/cancel`
**Descrição**: Cancelar assinatura
**Acesso**: Autenticado (proprietário da assinatura)

```javascript
// Payload de exemplo
{
  "cancelReason": "Não preciso mais do serviço",
  "cancel_at_period_end": true, // cancelar apenas no final do período atual
  "feedback": {
    "rating": 4,
    "comment": "Serviço bom, mas não preciso mais"
  }
}
```

**Response Success**:
```json
{
  "success": true,
  "message": "Assinatura cancelada com sucesso",
  "data": {
    "id": "subscription-789",
    "status": "cancelled",
    "canceled_at": "2024-01-15T10:00:00Z",
    "cancel_reason": "Não preciso mais do serviço",
    "cancel_at_period_end": true,
    "current_period_end": "2024-02-08T12:00:00Z",
    "access_until": "2024-02-08T12:00:00Z"
  }
}
```

#### `PUT /api/subscriptions/:id/reactivate`
**Descrição**: Reativar assinatura cancelada
**Acesso**: Autenticado (proprietário da assinatura)

**Response Success**:
```json
{
  "success": true,
  "message": "Assinatura reativada com sucesso",
  "data": {
    "id": "subscription-789",
    "status": "active",
    "reactivated_at": "2024-01-20T14:00:00Z",
    "current_period_start": "2024-01-20T14:00:00Z",
    "current_period_end": "2024-02-20T14:00:00Z",
    "next_payment_date": "2024-02-20T14:00:00Z"
  }
}
```

### 3. Estatísticas e Analytics

#### `GET /api/subscriptions/:id/stats`
**Descrição**: Obter estatísticas detalhadas de uma assinatura
**Acesso**: Autenticado (proprietário) ou Admin

**Response Success**:
```json
{
  "success": true,
  "data": {
    "subscription_id": "subscription-789",
    "overview": {
      "total_payments": 5,
      "total_paid": 299.50,
      "average_payment": 59.90,
      "subscription_age_days": 150,
      "next_payment_in_days": 15
    },
    "payment_history": [
      {
        "payment_date": "2024-01-08",
        "amount": 59.90,
        "status": "paid",
        "payment_method": "credit_card"
      },
      {
        "payment_date": "2024-02-08",
        "amount": 59.90,
        "status": "paid",
        "payment_method": "credit_card"
      }
    ],
    "usage_stats": {
      "features_used": [
        "produtos_ilimitados",
        "analytics_avancadas",
        "api_completa"
      ],
      "api_calls_last_month": 1247,
      "products_created": 25,
      "sales_volume": 12450.80
    },
    "health": {
      "payment_success_rate": 100,
      "avg_days_between_payments": 30,
      "failed_payments": 0,
      "churn_risk": "low"
    }
  }
}
```

### 4. Processamento de Pagamentos

#### `POST /api/subscriptions/payments/:id/process`
**Descrição**: Processar pagamento manualmente (admin)
**Acesso**: Admin com permissão financeira

**Response Success**:
```json
{
  "success": true,
  "message": "Pagamento processado com sucesso",
  "data": {
    "payment_id": "payment-102",
    "subscription_id": "subscription-789",
    "amount": 59.90,
    "status": "paid",
    "processed_at": "2024-01-20T16:00:00Z",
    "processed_by": "admin-001",
    "transaction_id": "txn_abc123"
  }
}
```

#### `POST /api/subscriptions/cron/process-payments`
**Descrição**: Processar todos os pagamentos pendentes (cron job)
**Acesso**: Admin com permissão de configurações da plataforma

**Response Success**:
```json
{
  "success": true,
  "message": "Processamento em lote executado com sucesso",
  "data": {
    "processed_payments": 45,
    "successful_payments": 42,
    "failed_payments": 3,
    "total_amount": 2695.50,
    "execution_time_ms": 1250,
    "failed_payment_details": [
      {
        "subscription_id": "subscription-101",
        "reason": "Cartão expirado",
        "amount": 49.90
      },
      {
        "subscription_id": "subscription-205",
        "reason": "Saldo insuficiente",
        "amount": 99.90
      }
    ]
  }
}
```

### 5. Configurações do Sistema

#### `GET /api/subscriptions/intervals`
**Descrição**: Listar intervalos de cobrança disponíveis
**Acesso**: Producer, Admin

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "id": "daily",
      "name": "Diário",
      "description": "Cobrança a cada dia"
    },
    {
      "id": "weekly",
      "name": "Semanal",
      "description": "Cobrança a cada semana"
    },
    {
      "id": "monthly",
      "name": "Mensal",
      "description": "Cobrança a cada mês"
    },
    {
      "id": "quarterly",
      "name": "Trimestral",
      "description": "Cobrança a cada 3 meses"
    },
    {
      "id": "yearly",
      "name": "Anual",
      "description": "Cobrança a cada ano"
    }
  ]
}
```

#### `GET /api/subscriptions/status-options`
**Descrição**: Listar opções de status de assinaturas
**Acesso**: Producer, Admin

**Response Success**:
```json
{
  "success": true,
  "data": [
    {
      "id": "trial",
      "name": "Período de Teste",
      "color": "#17a2b8"
    },
    {
      "id": "active",
      "name": "Ativo",
      "color": "#28a745"
    },
    {
      "id": "past_due",
      "name": "Em Atraso",
      "color": "#ffc107"
    },
    {
      "id": "cancelled",
      "name": "Cancelado",
      "color": "#dc3545"
    },
    {
      "id": "expired",
      "name": "Expirado",
      "color": "#6c757d"
    }
  ]
}
```

## Integração Frontend

### Componente de Assinatura
```jsx
import React, { useState, useEffect } from 'react';

const SubscriptionManagement = ({ userId, token }) => {
  const [subscriptions, setSubscriptions] = useState([]);
  const [plans, setPlans] = useState([]);
  const [loading, setLoading] = useState(true);
  const [selectedPlan, setSelectedPlan] = useState(null);
  
  useEffect(() => {
    loadSubscriptions();
    loadPlans();
  }, []);
  
  const loadSubscriptions = async () => {
    try {
      const response = await fetch('/api/subscriptions/my', {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setSubscriptions(data.data);
    } catch (error) {
      console.error('Erro ao carregar assinaturas:', error);
    }
  };
  
  const loadPlans = async () => {
    try {
      const response = await fetch('/api/subscriptions/plans', {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      const data = await response.json();
      setPlans(data.data);
    } catch (error) {
      console.error('Erro ao carregar planos:', error);
    } finally {
      setLoading(false);
    }
  };
  
  const createSubscription = async (planId, paymentMethod) => {
    try {
      const response = await fetch('/api/subscriptions', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({
          planId,
          paymentMethod
        })
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Assinatura criada com sucesso!');
        loadSubscriptions();
      }
    } catch (error) {
      alert('Erro ao criar assinatura: ' + error.message);
    }
  };
  
  const cancelSubscription = async (subscriptionId, reason) => {
    if (!confirm('Tem certeza que deseja cancelar esta assinatura?')) return;
    
    try {
      const response = await fetch(`/api/subscriptions/${subscriptionId}/cancel`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({
          cancelReason: reason,
          cancel_at_period_end: true
        })
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Assinatura cancelada com sucesso!');
        loadSubscriptions();
      }
    } catch (error) {
      alert('Erro ao cancelar assinatura: ' + error.message);
    }
  };
  
  const reactivateSubscription = async (subscriptionId) => {
    try {
      const response = await fetch(`/api/subscriptions/${subscriptionId}/reactivate`, {
        method: 'PUT',
        headers: { 'Authorization': `Bearer ${token}` }
      });
      
      const data = await response.json();
      if (data.success) {
        alert('Assinatura reativada com sucesso!');
        loadSubscriptions();
      }
    } catch (error) {
      alert('Erro ao reativar assinatura: ' + error.message);
    }
  };
  
  if (loading) return <div>Carregando assinaturas...</div>;
  
  return (
    <div className="subscription-management">
      <h2>🔄 Minhas Assinaturas</h2>
      
      {/* Lista de Assinaturas Ativas */}
      <div className="active-subscriptions">
        <h3>Assinaturas Ativas</h3>
        {subscriptions.filter(s => ['trial', 'active'].includes(s.status)).map(subscription => (
          <div key={subscription.id} className="subscription-card">
            <div className="subscription-header">
              <h4>{subscription.plan.name}</h4>
              <span className={`status ${subscription.status}`}>
                {getStatusLabel(subscription.status)}
              </span>
            </div>
            
            <div className="subscription-details">
              <p><strong>Valor:</strong> R$ {subscription.amount.toFixed(2)}</p>
              <p><strong>Próximo pagamento:</strong> {new Date(subscription.next_payment_date).toLocaleDateString()}</p>
              <p><strong>Intervalo:</strong> {subscription.plan.interval}</p>
            </div>
            
            <div className="subscription-actions">
              <button 
                onClick={() => viewStats(subscription.id)}
                className="btn btn-secondary"
              >
                📊 Ver Estatísticas
              </button>
              <button 
                onClick={() => cancelSubscription(subscription.id, 'Cancelamento voluntário')}
                className="btn btn-danger"
              >
                ❌ Cancelar
              </button>
            </div>
          </div>
        ))}
      </div>
      
      {/* Assinaturas Canceladas */}
      <div className="cancelled-subscriptions">
        <h3>Assinaturas Canceladas</h3>
        {subscriptions.filter(s => s.status === 'cancelled').map(subscription => (
          <div key={subscription.id} className="subscription-card cancelled">
            <div className="subscription-header">
              <h4>{subscription.plan.name}</h4>
              <span className="status cancelled">Cancelada</span>
            </div>
            
            <div className="subscription-details">
              <p><strong>Cancelada em:</strong> {new Date(subscription.canceled_at).toLocaleDateString()}</p>
              <p><strong>Acesso até:</strong> {new Date(subscription.access_until).toLocaleDateString()}</p>
            </div>
            
            <div className="subscription-actions">
              <button 
                onClick={() => reactivateSubscription(subscription.id)}
                className="btn btn-primary"
              >
                🔄 Reativar
              </button>
            </div>
          </div>
        ))}
      </div>
      
      {/* Planos Disponíveis */}
      <div className="available-plans">
        <h3>➕ Criar Nova Assinatura</h3>
        <div className="plans-grid">
          {plans.map(plan => (
            <div key={plan.id} className="plan-card">
              <h4>{plan.name}</h4>
              <p className="price">R$ {plan.price.toFixed(2)}/{plan.interval}</p>
              <ul className="features">
                {plan.features.map((feature, index) => (
                  <li key={index}>{feature}</li>
                ))}
              </ul>
              <button 
                onClick={() => setSelectedPlan(plan)}
                className="btn btn-primary"
              >
                Assinar
              </button>
            </div>
          ))}
        </div>
      </div>
      
      {/* Modal de pagamento */}
      {selectedPlan && (
        <PaymentModal 
          plan={selectedPlan}
          onConfirm={(paymentMethod) => {
            createSubscription(selectedPlan.id, paymentMethod);
            setSelectedPlan(null);
          }}
          onCancel={() => setSelectedPlan(null)}
        />
      )}
    </div>
  );
  
  const getStatusLabel = (status) => {
    const labels = {
      trial: 'Teste',
      active: 'Ativo',
      past_due: 'Em Atraso',
      cancelled: 'Cancelado',
      expired: 'Expirado'
    };
    return labels[status] || status;
  };
  
  const viewStats = (subscriptionId) => {
    // Implementar navegação para página de estatísticas
    window.location.href = `/subscriptions/${subscriptionId}/stats`;
  };
};
```

### Hook para Assinaturas
```jsx
const useSubscriptions = () => {
  const [subscriptions, setSubscriptions] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const createSubscription = async (planId, paymentMethod, token) => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch('/api/subscriptions', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({
          planId,
          paymentMethod
        })
      });
      
      const data = await response.json();
      
      if (!data.success) {
        throw new Error(data.message);
      }
      
      return data.data;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  const getMySubscriptions = async (token, status = null) => {
    setLoading(true);
    setError(null);
    
    try {
      const url = status ? `/api/subscriptions/my?status=${status}` : '/api/subscriptions/my';
      const response = await fetch(url, {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      
      const data = await response.json();
      
      if (data.success) {
        setSubscriptions(data.data);
        return data.data;
      } else {
        throw new Error(data.message);
      }
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  const cancelSubscription = async (subscriptionId, reason, token) => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch(`/api/subscriptions/${subscriptionId}/cancel`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`
        },
        body: JSON.stringify({
          cancelReason: reason,
          cancel_at_period_end: true
        })
      });
      
      const data = await response.json();
      
      if (!data.success) {
        throw new Error(data.message);
      }
      
      return data.data;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  const getSubscriptionStats = async (subscriptionId, token) => {
    try {
      const response = await fetch(`/api/subscriptions/${subscriptionId}/stats`, {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      
      const data = await response.json();
      
      if (data.success) {
        return data.data;
      } else {
        throw new Error(data.message);
      }
    } catch (err) {
      setError(err.message);
      throw err;
    }
  };
  
  return {
    subscriptions,
    loading,
    error,
    createSubscription,
    getMySubscriptions,
    cancelSubscription,
    getSubscriptionStats
  };
};
```

## Códigos de Erro

| Código | Descrição |
|--------|-----------|
| 400 | Parâmetros obrigatórios ausentes ou inválidos |
| 401 | Token de autenticação inválido |
| 403 | Acesso negado (permissões insuficientes) |
| 404 | Plano ou assinatura não encontrada |
| 500 | Erro interno do servidor |

## Considerações de Segurança

1. **Validação de Propriedade**: Verificar se assinatura pertence ao usuário
2. **Permissões Granulares**: Controle de acesso por função
3. **Logs de Auditoria**: Registro de todas as operações
4. **Validação de Pagamentos**: Verificação de dados de cartão
5. **Rate Limiting**: Proteção contra uso excessivo

## Melhores Práticas

1. **Períodos de Teste**: Oferecer trials para aumentar conversão
2. **Cancelamento Gracioso**: Manter acesso até o final do período pago
3. **Comunicação Proativa**: Notificar sobre pagamentos e mudanças
4. **Flexibilidade**: Permitir pausar/reativar assinaturas
5. **Analytics**: Monitorar métricas de churn e LTV

Este sistema de assinaturas oferece uma base sólida para monetização recorrente, com gestão completa do ciclo de vida das assinaturas e processamento automatizado de pagamentos.
